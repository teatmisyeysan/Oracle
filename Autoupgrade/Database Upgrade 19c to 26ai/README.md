# Oracle Database 19c to 26ai Upgrade Using AutoUpgrade
## 1.  Prepare the AutoUpgrade JAR
```bash
cd /u01/app/oracle/product/23.26.3/dbhome_1/rdbms/admin
mv autoupgrade.jar autoupgrade.jar.bkp
cp /u01/app/oracle/autoupgrade/autoupgrade.jar .
```

## 2. Validate Autoupgrad Version
```bash
[oracle@ms-vm-01 admin]$ java -jar autoupgrade.jar -version
build.version 26.5.260807
build.date 2026/08/07 17:06:01 +0000
build.hash 0d4f2f519
build.hash_date 2026/07/31 15:26:55 +0000
build.supported_target_versions 12.2,18,19,21,23,26
build.type production
build.label (HEAD, tag: v26.5, origin/stable_devel, stable_devel)
build.MOS_NOTE KB123450
build.MOS_LINK https://support.oracle.com/support/?anchorId=&kmContentId=2485457&page=sptemplate&sptemplate=km-article

[oracle@ms-vm-01 admin]$
```

### 3. Generate a Sample Config File
Use the Java bundled with the target Oracle 26ai home:
```bash
$ORACLE_BASE/product/23.26.3/dbhome_1/jdk/bin/java \
-jar $ORACLE_BASE/product/23.26.3/dbhome_1/rdbms/admin/autoupgrade.jar \
-create_sample_file config /tmp/config.cfg
```

```bash
* Or we can custom config "au-upgrade.cfg"

upg1.log_dir=/u01/app/oracle/autoupgrade/logs
upg1.sid=PROD19C
upg1.source_home=/u01/app/oracle/product/19.0.0/dbhome_1
upg1.target_home=/u01/app/oracle/product/23.26.3/dbhome_1
upg1.start_time=NOW
upg1.upgrade_node=ms-vm-01.localdomain
upg1.run_utlrp=yes
upg1.timezone_upg=yes
upg1.target_version=26
upg1.restoration=no
```
## 4. Run AutoUpgrade in ANALYZE Mode
```bash
java -jar /u01/app/oracle/product/23.26.3/dbhome_1/rdbms/admin/autoupgrade.jar -config /u01/app/oracle/autoupgrade/etc/au-upgrade.cfg -mode analyze

[oracle@ms-vm-01 etc]$ java -jar /u01/app/oracle/product/23.26.3/dbhome_1/rdbms/admin/autoupgrade.jar -config /u01/app/oracle/autoupgrade/etc/au-upgrade.cfg -mode analyze
AutoUpgrade 26.5.260807 launched with default internal options
Processing config file ...
Loading AutoUpgrade keystore
AutoUpgrade keystore is loaded
Pluggable database PDB in PROD19C is MOUNTED and it will not be processed
+--------------------------------+
| Starting AutoUpgrade execution |
+--------------------------------+
1 CDB(s) plus 1 PDB(s) will be analyzed
Type 'help' to list console commands
upg> Job 100 completed
------------------- Final Summary --------------------
Number of databases            [ 1 ]

Jobs finished                  [1]
Jobs failed                    [0]

Please check the summary report at:
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.html
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.log
[oracle@ms-vm-01 etc]$
```

* Open PDB
```bash
sqlplus / as sysdb
show pdbs;

[oracle@ms-vm-01 etc]$ sql

SQL*Plus: Release 19.0.0.0.0 - Production on Wed Aug 19 14:57:13 2026
Version 19.30.0.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.30.0.0.0

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB                            MOUNTED
SQL> alter pluggable database pdb open;

Pluggable database altered.

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB                            READ WRITE NO
SQL>
```
<img width="921" height="615" alt="image" src="https://github.com/user-attachments/assets/b69f3e3d-7d6c-454a-82c1-3bee98a22e00" />

* Check report failed and fixed with reclaim space on +RECO

* Pre-check again
```bash
[oracle@ms-vm-01 etc]$ java -jar /u01/app/oracle/product/23.26.3/dbhome_1/rdbms/admin/autoupgrade.jar -config /u01/app/oracle/autoupgrade/etc/au-upgrade.cfg -mode analyze
AutoUpgrade 26.5.260807 launched with default internal options
Processing config file ...
Loading AutoUpgrade keystore
AutoUpgrade keystore is loaded
+--------------------------------+
| Starting AutoUpgrade execution |
+--------------------------------+
1 CDB(s) plus 2 PDB(s) will be analyzed
Type 'help' to list console commands
upg> 
upg> lsj
+----+-------+---------+---------+-------+----------+-------+----------------------------+
|Job#|DB_NAME|    STAGE|OPERATION| STATUS|START_TIME|UPDATED|                     MESSAGE|
+----+-------+---------+---------+-------+----------+-------+----------------------------+
| 101|PROD19C|PRECHECKS|EXECUTING|RUNNING|  15:08:05| 1s ago|Loading database information|
+----+-------+---------+---------+-------+----------+-------+----------------------------+
Total jobs 1

upg> tasks
+---+--------------+-------------+
| ID|          NAME|         Job#|
+---+--------------+-------------+
|  1|          main|      WAITING|
| 10|Common-Cleaner|TIMED_WAITING|
| 77|    event_loop|TIMED_WAITING|
| 78|       console|     RUNNABLE|
| 79|  queue_reader|      WAITING|
| 81|         cmd-0|      WAITING|
| 82|       StatUpg|      WAITING|
| 83|    event_loop|TIMED_WAITING|
| 85| job_manager-0|TIMED_WAITING|
| 92|     exec_loop|      WAITING|
|296|prod19c-puic-0|      WAITING|
|297|prod19c-puic-1|      WAITING|
|298|prod19c-puic-2|      WAITING|
|299|prod19c-puic-3|      WAITING|
|300|prod19c-puic-4|      WAITING|
|301|prod19c-puic-5|      WAITING|
|302|prod19c-puic-6|      WAITING|
|303|prod19c-puic-7|      WAITING|
|345| sql-[10C666] |     RUNNABLE|
|349| sql-[5B94EE] |     RUNNABLE|
|350| sql-[CB9744] |     RUNNABLE|
|351| sql-[2AB196] |     RUNNABLE|
|352| sql-[BCCCE4] |     RUNNABLE|
|353| sql-[EAAB77] |     RUNNABLE|
|354| sql-[3FB5FB] |     RUNNABLE|
|355| sql-[2AC5A9] |     RUNNABLE|
+---+--------------+-------------+
upg> Job 101 completed
------------------- Final Summary --------------------
Number of databases            [ 1 ]

Jobs finished                  [1]
Jobs failed                    [0]

Please check the summary report at:
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.html
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.log
[oracle@ms-vm-01 etc]$
```

<img width="924" height="574" alt="image" src="https://github.com/user-attachments/assets/dab19c26-e25f-4498-af39-6fee99bdf775" />
