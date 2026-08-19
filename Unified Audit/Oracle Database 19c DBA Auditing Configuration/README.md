# Step-by-Step Oracle 19c DBA Auditing Configuration
## 1. Check Current Auditing Status
```bash
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Unified Auditing';
SHOW PARAMETER audit_trail;
```

## 2. Enable Unified Auditing (If Not Enabled)
### A. Shutdown database.
```bash
sqlplus / as sysdba
shut immediate;
```

### B. Relink Oracle Binary with Unified Auditing Enabled
```bash
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk uniaud_on ioracle
```
### C. Start database.
```bash
sqlplus / as sysdba
startup;
```

### D. Verify Unified Auditing Status.
```bash
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Unified Auditing';
```


## 3. Create Audit Policies for DBA Privileges
Oracle 19c Unified Auditing allows you to group multiple audit rules into audit policies. These policies help track who grants DBA power, who uses it, and when privileged access occurs.

### A. Audit When DBA Role Is Granted or Revoked.
```bash
-- create
CREATE AUDIT POLICY audit_dba_role_grants
ACTIONS
GRANT,
REVOKE
ROLES DBA;

-- Now enable the Policy (audit_dba_role_grants)
AUDIT POLICY audit_dba_role_grants;
```
### B. Audit All DBA Privilege Usage.
```bash
-- Create policy
CREATE AUDIT POLICY audit_dba_activities
PRIVILEGES
ALTER SYSTEM,
ALTER DATABASE,
CREATE USER,
DROP USER,
ALTER USER,
CREATE ROLE,
DROP ANY TABLE,
DELETE ANY TABLE,
DROP ANY PROCEDURE,
GRANT ANY PRIVILEGE,
GRANT ANY ROLE;

Note: - This policy audits actual usage of powerful DBA-level privileges, not just role assignment.

-- Now enable the Policy (audit_dba_activities)
AUDIT POLICY audit_dba_activities;
```

### C. Audit SYSDBA / SYSOPER Connections.
SYSDBA and SYSOPER are the highest privilege levels in Oracle. Auditing these connections is mandatory for security and compliance.
```bash
-- Create policy for privileged connections
CREATE AUDIT POLICY audit_privileged_connections ACTIONS LOGON;

-- Enable Policy for DBA-Level Users (SYSDBA and SYSOPER)
AUDIT POLICY audit_privileged_connections BY USERS WITH GRANTED ROLES DBA;

Note: - It will Applies only to users with DBA role.
```
### D. Audit SYS Operations (OS-Authenticated Actions)
This parameter controls auditing of SYSDBA actions performed outside the database like Startup / Shutdown, Recovery, OS-authenticated SYS connections. It is enabled by default, if not then we can enable it using below commands
```bash
SHOW PARAMETER audit_sys_operations;

Note: - If it is not enable you can use below command to enable it which also requires database reboot.

ALTER SYSTEM SET audit_sys_operations=TRUE SCOPE=SPFILE;
```

### E. How to View These Audit Records
```bash
-- You can view these records using below command.

set lines 999 pages 999
COL EVENT_TIMESTAMP for a40
COL DBUSERNAME for a20
COL ACTION_NAME for a20
COL CLIENT_PROGRAM_NAME for a50
COL USERHOST for a25
SELECT
  EVENT_TIMESTAMP,
  DBUSERNAME,
  ACTION_NAME,
  USERHOST,
  CLIENT_PROGRAM_NAME
FROM UNIFIED_AUDIT_TRAIL
ORDER BY EVENT_TIMESTAMP DESC;
```
## 4. Create Read-Only Role for Non-Privileged Users
### A read-only role is used for users who only need to view data (reports, dashboards, audits) and must not change anything in the database.
```bash
-- Create a Read-Only Role.
CREATE ROLE readonly_user;

Note: - Above command will help to creates a container role to hold all read-only permissions
```
### B. Grant SELECT on Specific Schema Objects.
```bash
BEGIN
  FOR t IN (SELECT table_name FROM dba_tables WHERE owner = 'HR') LOOP
    EXECUTE IMMEDIATE 
      'GRANT SELECT ON HR.' || t.table_name || ' TO readonly_user';
  END LOOP;
END;
/
```
### C. Grant SELECT ANY TABLE (more permissive)
```bash
GRANT SELECT ANY TABLE TO readonly_user;
```
### D. Grant Basic Session Privilege
```bash
GRANT CREATE SESSION TO readonly_user;

Note: - Above command allows users to log in to the database because Without this, the user cannot connect, even with SELECT privileges
```
### E. Create read-only user
```bash
CREATE USER report_user IDENTIFIED BY "SecurePass123#";
```
### F. Assign the Read-Only Role to the User
```bash
GRANT readonly_user TO report_user;
```
## 5. Audit Policy for Monitoring Read-Only Users
Sometimes even read-only users can become a security risk like Privileges are granted by mistake, Roles are misconfigured, Applications attempt write operations, Someone tries to misuse access intentionally.

This audit policy helps you detect and prove such violations.

### A. Create Audit Policy for Read-Only Violations.
```bash
CREATE AUDIT POLICY audit_readonly_violations
ACTIONS
INSERT,
UPDATE,
DELETE,
CREATE TABLE,
DROP TABLE,
ALTER TABLE;
```
### B. Enable the Policy Only for Read-Only Users
```bash
AUDIT POLICY audit_readonly_violations BY USERS WITH GRANTED ROLES readonly_user;
```
### 6. Viewing Unified Audit Records
### A. View All Unified Audit Records (Last 7 Days)
```bash
set lines 999 pages 999
col USERHOST for a20
col OS_USERNAME for a20
col SQL_TEXT for a40
SELECT
EVENT_TIMESTAMP,
DBUSERNAME,
ACTION_NAME,
OBJECT_SCHEMA,
OBJECT_NAME,
SQL_TEXT,
RETURN_CODE,
OS_USERNAME,
USERHOST
FROM UNIFIED_AUDIT_TRAIL
WHERE EVENT_TIMESTAMP > SYSDATE - 7
ORDER BY EVENT_TIMESTAMP DESC;
```

### B. View DBA privilege usage
```bash
set lines 999 pages 999
COL EVENT_TIMESTAMP for a40
COL DBUSERNAME for a20
COL ACTION_NAME for a20
col SYSTEM_PRIVILEGE_USED for a30
SELECT
EVENT_TIMESTAMP,
DBUSERNAME,
ACTION_NAME,
SYSTEM_PRIVILEGE_USED,
RETURN_CODE
FROM UNIFIED_AUDIT_TRAIL
WHERE SYSTEM_PRIVILEGE_USED IS NOT NULL
ORDER BY EVENT_TIMESTAMP DESC;
```


### C. View failed login attempts
```bash
set lines 999 pages 999
COL EVENT_TIMESTAMP for a40
col USERHOST for a30
col OS_USERNAME for a20
COL DBUSERNAME for a20
SELECT
EVENT_TIMESTAMP,
DBUSERNAME,
OS_USERNAME,
USERHOST,
RETURN_CODE
FROM UNIFIED_AUDIT_TRAIL
WHERE ACTION_NAME = 'LOGON'
AND RETURN_CODE != 0
ORDER BY EVENT_TIMESTAMP DESC;
```

## 7. Set the Audit Retention Window
```bash
BEGIN
  DBMS_AUDIT_MGMT.SET_LAST_ARCHIVE_TIMESTAMP(
    AUDIT_TRAIL_TYPE => DBMS_AUDIT_MGMT.AUDIT_TRAIL_UNIFIED,
    LAST_ARCHIVE_TIME => SYSTIMESTAMP - 90
  );
END;
/

Note: - Above command will help to clear Audit records older than 90 days which are eligible for cleanup.
```

## 8. Create an Automatic Cleanup (Purge) Job
```bash
BEGIN
  DBMS_AUDIT_MGMT.CREATE_PURGE_JOB(
    AUDIT_TRAIL_TYPE => DBMS_AUDIT_MGMT.AUDIT_TRAIL_UNIFIED,
    AUDIT_TRAIL_PURGE_INTERVAL => 24, -- hours
    AUDIT_TRAIL_PURGE_NAME => 'DAILY_AUDIT_PURGE',
    USE_LAST_ARCH_TIMESTAMP => TRUE
  );
END;
/

Note: - Above command will help to creates a scheduled job which runs every 24 hours and delete only audit records  older than 90 days.
```
## 9. Disable Unified Auditing in Oracle 19c
### A. Shutdown the database.
```bash
sqlplus / as sysdba
SHUTDOWN IMMEDIATE;
EXIT;
```
### B. Relink Oracle Binary to Disable Unified Auditing.
```bash
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk uniaud_off ioracle
```
### C. Start the Database.
```bash
sqlplus / as sysdba
STARTUP;
```
### D. Verify Unified Auditing Is Disabled.
```bash
SELECT VALUE
FROM V$OPTION
WHERE PARAMETER = 'Unified Auditing';
```
