# Step-by-Step Oracle 19c DBA Auditing Configuration
## 1. Check Current Auditing Status
```bash
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Unified Auditing';
SHOW PARAMETER audit_trail;
```

## 2. Enable Unified Auditing (If Not Enabled)
### a) Shutdown database.
```bash
sqlplus / as sysdba
shut immediate;
```

### b) Relink Oracle Binary with Unified Auditing Enabled
```bash
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk uniaud_on ioracle
```
### c) Start database.
```bash
sqlplus / as sysdba
startup;
```

### d) Verify Unified Auditing Status.
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
