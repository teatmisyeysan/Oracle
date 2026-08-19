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
