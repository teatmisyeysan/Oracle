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

## 3. Generate a Sample Config File
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

Check report failed and fixed with reclaim space on +RECO
Pre-check again
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

## 5. Start the Upgrade (DEPLOY Mode)
```bash
[oracle@ms-vm-01 etc]$ java -jar /u01/app/oracle/product/23.26.3/dbhome_1/rdbms/admin/autoupgrade.jar -config /u01/app/oracle/autoupgrade/etc/au-upgrade.cfg -mode deploy
Previous execution found loading latest data
Total jobs recovered: 1
Loading AutoUpgrade keystore
AutoUpgrade keystore is loaded
+--------------------------------+
| Starting AutoUpgrade execution |
+--------------------------------+
Type 'help' to list console commands
upg> ls
Unrecognized cmd: ls
upg> lsj
+----+-------+---------+---------+-------+----------+-------+-------+
|Job#|DB_NAME|    STAGE|OPERATION| STATUS|START_TIME|UPDATED|MESSAGE|
+----+-------+---------+---------+-------+----------+-------+-------+
| 102|PROD19C|DBUPGRADE|EXECUTING|RUNNING|  15:13:25| 3s ago|Running|
+----+-------+---------+---------+-------+----------+-------+-------+
Total jobs 1

upg> lsj
+----+-------+---------+---------+-------+----------+-------+--------------------+
|Job#|DB_NAME|    STAGE|OPERATION| STATUS|START_TIME|UPDATED|             MESSAGE|
+----+-------+---------+---------+-------+----------+-------+--------------------+
| 102|PROD19C|DBUPGRADE|EXECUTING|RUNNING|  15:13:25|13s ago|23%Upgraded CDB$ROOT|
+----+-------+---------+---------+-------+----------+-------+--------------------+
Total jobs 1

upg> status -job 102
Details

        Job No           102
        Oracle SID       PROD19C
        Start Time       26/08/19 15:13:25
        Elapsed (min):   76
        End time:        N/A

Logfiles

        Logs Base:    /u01/app/oracle/autoupgrade/logs/PROD19C
        Job logs:     /u01/app/oracle/autoupgrade/logs/PROD19C/102
        Stage logs:   /u01/app/oracle/autoupgrade/logs/PROD19C/102/sysupdates
        TimeZone:     /u01/app/oracle/autoupgrade/logs/PROD19C/temp
        Remote Dirs:

Stages
        SETUP            <1 min
        PREUPGRADE       <1 min
        PRECHECKS        <1 min
        PREFIXUPS        4 min
        DRAIN            3 min
        DBUPGRADE        44 min
        DISPATCH         <1 min
        POSTCHECKS       <1 min
        DISPATCH         <1 min
        POSTFIXUPS       10 min
        POSTUPGRADE      <1 min
        SYSUPDATES       ~4 min (RUNNING)

Stage-Progress Per Container

        +--------+----------+
        |Database|SYSUPDATES|
        +--------+----------+
        |CDB$ROOT|     1  % |
        |PDB$SEED|     0  % |
        |     PDB|     0  % |
        +--------+----------+

upg> tasks
+--+--------------+-------------+
|ID|          NAME|         Job#|
+--+--------------+-------------+
| 1|          main|      WAITING|
|10|Common-Cleaner|TIMED_WAITING|
|18|    event_loop|TIMED_WAITING|
|19|       console|     RUNNABLE|
|20|  queue_reader|      WAITING|
|22|         cmd-0|      WAITING|
|23|       StatUpg|      WAITING|
|24|    event_loop|TIMED_WAITING|
|26| job_manager-0|TIMED_WAITING|
|33|     exec_loop|      WAITING|
|34|     exec_loop|      WAITING|
+--+--------------+-------------+
upg>
upg> Job 102 completed
------------------- Final Summary --------------------
Number of databases            [ 1 ]

Jobs finished                  [1]
Jobs failed                    [0]
Jobs restored                  [0]
Jobs pending                   [0]

Please check the summary report at:
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.html
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.log
[oracle@ms-vm-01 etc]$
```

## 6. Validate Log
```bash
[oracle@ms-vm-01 etc]$ cat  /u01/app/oracle/autoupgrade/logs/cfgtoollogs/upgrade/auto/status/status.log
==========================================
          Autoupgrade Summary Report
==========================================
[Date]           Wed Aug 19 16:30:44 ICT 2026
[Number of Jobs] 1
==========================================
[Job ID] 102
==========================================
[DB Name]                PROD19C
[Version Before Upgrade] 19.30.0.0.0
[Version After Upgrade]  23.26.3.0.0
------------------------------------------
[Stage Name]    PREUPGRADE
[Status]        SUCCESS
[Start Time]    2026-08-19 15:13:25
[Duration]      0:00:00
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/preupgrade
------------------------------------------
[Stage Name]    PRECHECKS
[Status]        SUCCESS
[Start Time]    2026-08-19 15:13:25
[Duration]      0:00:16
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/prechecks
[Detail]        /u01/app/oracle/autoupgrade/logs/PROD19C/102/prechecks/prod19c_preupgrade.log
                Check passed and no manual intervention needed
------------------------------------------
[Stage Name]    PREFIXUPS
[Status]        SUCCESS
[Start Time]    2026-08-19 15:13:41
[Duration]      0:04:38
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/prefixups
[Detail]        /u01/app/oracle/autoupgrade/logs/PROD19C/102/prefixups/prefixups.html
------------------------------------------
[Stage Name]    DRAIN
[Status]        SUCCESS
[Start Time]    2026-08-19 15:18:20
[Duration]      0:03:26
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/drain
------------------------------------------
[Stage Name]    DBUPGRADE
[Status]        SUCCESS
[Start Time]    2026-08-19 15:30:08
[Duration]      0:44:36
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/dbupgrade
------------------------------------------
[Stage Name]    POSTCHECKS
[Status]        SUCCESS
[Start Time]    2026-08-19 16:14:46
[Duration]      0:00:04
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/postchecks
[Detail]        /u01/app/oracle/autoupgrade/logs/PROD19C/102/postchecks/prod19c_postupgrade.log
                Check passed and no manual intervention needed
------------------------------------------
[Stage Name]    POSTFIXUPS
[Status]        SUCCESS
[Start Time]    2026-08-19 16:14:51
[Duration]      0:10:42
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/postfixups
[Detail]        /u01/app/oracle/autoupgrade/logs/PROD19C/102/postfixups/postfixups.html
------------------------------------------
[Stage Name]    POSTUPGRADE
[Status]        SUCCESS
[Start Time]    2026-08-19 16:25:34
[Duration]      0:00:01
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/postupgrade
------------------------------------------
[Stage Name]    SYSUPDATES
[Status]        SUCCESS
[Start Time]    2026-08-19 16:25:35
[Duration]      0:05:09
[Log Directory] /u01/app/oracle/autoupgrade/logs/PROD19C/102/sysupdates
------------------------------------------
Summary:/u01/app/oracle/autoupgrade/logs/PROD19C/102/dbupgrade/upg_summary.log
[oracle@ms-vm-01 etc]$
```
<img width="987" height="569" alt="image" src="https://github.com/user-attachments/assets/c6ed0649-802d-47b8-895a-2e335cc1f1c9" />

### 7. Post-Upgrade
```bash
[oracle@ms-vm-01 ~]$ . db26ai
[oracle@ms-vm-01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 23.26.3.0.0 - Production on Wed Aug 19 16:47:35 2026
Version 23.26.3.0.0

Copyright (c) 1982, 2026, Oracle.  All rights reserved.


Connected to:
Oracle AI Database 26ai Enterprise Edition Release 23.26.3.0.0 - Production
Version 23.26.3.0.0

SQL>
SQL> SELECT * FROM v$timezone_file;

FILENAME                VERSION     CON_ID
-------------------- ---------- ----------
timezlrg_45.dat              45          0

SQL> SET LINESIZE 200
COLUMN comp_name FORMAT A50
COLUMN version FORMAT A15
COLUMN status FORMAT A10

SELECT comp_name, version, status
FROM dba_registry
ORDER BY comp_name;
SQL> SQL> SQL> SQL> SQL>   2    3
COMP_NAME                                          VERSION         STATUS
-------------------------------------------------- --------------- ----------
JServer JAVA Virtual Machine                       23.0.0.0.0      VALID
OLAP Analytic Workspace                            23.0.0.0.0      VALID
Oracle Database Catalog Views                      23.0.0.0.0      VALID
Oracle Database Java Packages                      23.0.0.0.0      VALID
Oracle Database Packages and Types                 23.0.0.0.0      VALID
Oracle Database Vault                              23.0.0.0.0      VALID
Oracle Label Security                              23.0.0.0.0      VALID
Oracle OLAP API                                    23.0.0.0.0      VALID
Oracle Real Application Clusters                   23.0.0.0.0      OPTION OFF
Oracle Text                                        23.0.0.0.0      VALID
Oracle Workspace Manager                           23.0.0.0.0      VALID

COMP_NAME                                          VERSION         STATUS
-------------------------------------------------- --------------- ----------
Oracle XDK                                         23.0.0.0.0      VALID
Oracle XML Database                                23.0.0.0.0      VALID
Spatial                                            23.0.0.0.0      VALID

14 rows selected.

SQL>

```
```bash
SQL> SET LINESIZE 200
SQL> COLUMN action_time FORMAT A12
COLUMN action FORMAT A10
COLUMN status FORMAT A10
COLUMN description FORMAT A80
SQL> COLUMN patch_id FORMAT 99999999

SELECT TO_CHAR(action_time, 'YYYY-MM-DD') AS action_time,
SQL> SQL>        action,
       status,
       description,
       patch_SQL> id
FROM   sys.dba_registry_sqlpatch
ORDER  BY action_time;
SQL> SQL>   2    3    4    5    6    7
ACTION_TIME  ACTION     STATUS     DESCRIPTION                                                                       PATCH_ID
------------ ---------- ---------- -------------------------------------------------------------------------------- ---------
2026-08-19   APPLY      SUCCESS    DATAPUMP BUNDLE PATCH 23.26.3.0.0                                                 39593097
2026-08-19   APPLY      SUCCESS    Database Release Update : 23.26.3.0.0 (39578879) Gold Image                       39578879

SQL>
```
