
# Install a New Oracle home for Oracle AI Database 26ai
* Prerequisites
```bash
wget https://download.oracle.com/otn-pub/otn_software/autoupgrade.jar
```
```bash
mkdir autoupgrade
mkdir autoupgrade/logs
mkdir autoupgrade/patches
mkdir autoupgrade/keystore
mkdir autoupgrade/etc
mkdir -p /u01/app/oracle/product/23.26.3/dbhome_1
```
```bash
vi au-download.cfg
global.global_log_dir=/u01/app/oracle/autoupgrade/logs
global.keystore=/u01/app/oracle/autoupgrade/keystore
dl.folder=/u01/app/oracle/autoupgrade/patches
dl.patch=RECOMMENDED
dl.target_version=23
dl.platform=LINUX.X64
[oracle@ms-vm-01 etc]$
```
```bash
[oracle@ms-vm-01 etc]$ vi au-create-home.cfg
global.global_log_dir=/u01/app/oracle/autoupgrade/logs
global.keystore=/u01/app/oracle/autoupgrade/keystore
crh.folder=/u01/app/oracle/autoupgrade/patches
crh.patch=RECOMMENDED
crh.target_version=26
crh.source_home=/u01/app/oracle/product/19.0.0/dbhome_1
crh.platform=LINUX.X64
crh.target_home=/u01/app/oracle/product/23.26.3/dbhome_1
crh.home_settings.edition=EE
crh.home_settings.oracle_base=/u01/app/oracle
crh.home_settings.inventory_location=/u01/app/oraInventory
crh.upgrade_node=ms-vm-01
crh.download=no
[oracle@ms-vm-01 etc]$
```
## 1. Load Credential 
```bash
[oracle@ms-vm-01 autoupgrade]$ java -jar autoupgrade.jar -config etc/au-download.cfg -patch -load_password
Processing config file ...
Loading AutoUpgrade Patching keystore
AutoUpgrade Patching keystore is loaded

Starting AutoUpgrade Patching Password Loader - Type help for available options

MOS> exit

AutoUpgrade Patching Password Loader finished - Exiting AutoUpgrade Patching
[oracle@ms-vm-01 autoupgrade]$

```

## 2. Download RU Patch
```bash
[oracle@ms-vm-01 autoupgrade]$ java -jar autoupgrade.jar -config etc/au-download.cfg -patch -mode download
AutoUpgrade Patching 26.5.260807 launched with default internal options
Processing config file ...
Loading AutoUpgrade Patching keystore
AutoUpgrade Patching keystore is loaded

Connected to MOS - Searching for specified patches

--------------------------------------------------------
Downloading files to /u01/app/oracle/autoupgrade/patches
--------------------------------------------------------
DATABASE RELEASE UPDATE 23.26.3.0.0(GOLD IMAGE)
    File: p39581612_230000_Linux-x86-64.zip - VALIDATED

OPatch 12.2.0.1.52 for DB 23.0.0.0.0 (Aug 2026)
    File: p6880880_230000_Linux-x86-64.zip - VALIDATED

DATAPUMP BUNDLE PATCH 23.26.3.0.0
    File: p39593097_2326300DBRU_Generic.zip - VALIDATED

DATABASE MRP 23.26.3.0.0 AUG 2026
    File: p39834059_2326300DBRU_Linux-x86-64.zip - VALIDATED

autoupgrade.jar 26.5 (August 2026)
    File: autoupgrade.jar - VALIDATED
```

## 3. Create new home
```bash
[oracle@ms-vm-01 autoupgrade]$ java -jar autoupgrade.jar -config etc/au-create-home.cfg -patch -mode create_home              AutoUpgrade Patching 26.5.260807 launched with default internal options
Processing config file ...
Loading AutoUpgrade Patching keystore
AutoUpgrade Patching keystore is loaded
+-----------------------------------------+
| Starting AutoUpgrade Patching execution |
+-----------------------------------------+
Type 'help' to list console commands
patch> ls
Unrecognized cmd: ls
patch> lsj
+----+-------------+-------+---------+-------+----------+-------+---------------------+
|Job#|      DB_NAME|  STAGE|OPERATION| STATUS|START_TIME|UPDATED|              MESSAGE|
+----+-------------+-------+---------+-------+----------+-------+---------------------+
| 100|create_home_1|EXTRACT|EXECUTING|RUNNING|  13:36:30| 8s ago|Extracting Gold Image|
+----+-------------+-------+---------+-------+----------+-------+---------------------+
Total jobs 1

patch>
patch> status -job 100
Details

        Job No           100
        Oracle SID       create_home_1
        Start Time       26/08/19 13:36:30
        Elapsed (min):   1
        End time:        N/A

Logfiles

        Logs Base:    /u01/app/oracle/autoupgrade/logs/create_home_1
        Job logs:     /u01/app/oracle/autoupgrade/logs/create_home_1/100
        Stage logs:   /u01/app/oracle/autoupgrade/logs/create_home_1/100/install
        TimeZone:     /u01/app/oracle/autoupgrade/logs/create_home_1/temp
        Remote Dirs:

Stages
        PENDING          <1 min
        PREACTIONS       <1 min
        EXTRACT          1 min
        DBTOOLS          <1 min
        INSTALL          ~0 min (RUNNING)
        OH_PATCHING
        OPTIONS
        ROOTSH
        POSTACTIONS

Stage-Progress Per Container

        The Stage INSTALL does not have any data to show
patch>
patch> status -job 100
Details

        Job No           100
        Oracle SID       create_home_1
        Start Time       26/08/19 13:36:30
        Elapsed (min):   4
        End time:        N/A

Logfiles

        Logs Base:    /u01/app/oracle/autoupgrade/logs/create_home_1
        Job logs:     /u01/app/oracle/autoupgrade/logs/create_home_1/100
        Stage logs:   /u01/app/oracle/autoupgrade/logs/create_home_1/100/opatch
        TimeZone:     /u01/app/oracle/autoupgrade/logs/create_home_1/temp
        Remote Dirs:

Stages
        PENDING          <1 min
        PREACTIONS       <1 min
        EXTRACT          1 min
        DBTOOLS          <1 min
        INSTALL          2 min
        OH_PATCHING      ~1 min (RUNNING)
        OPTIONS
        ROOTSH
        POSTACTIONS

Stage-Progress Per Container

        The Stage OH_PATCHING does not have any data to show
patch> lsj
+----+-------------+-----------+---------+-------+----------+-------+---------------------------------+
|Job#|      DB_NAME|      STAGE|OPERATION| STATUS|START_TIME|UPDATED|                          MESSAGE|
+----+-------------+-----------+---------+-------+----------+-------+---------------------------------+
| 100|create_home_1|OH_PATCHING|EXECUTING|RUNNING|  13:36:30|35s ago|DATABASE MRP 23.26.3.0.0 AUG 2026|
+----+-------------+-----------+---------+-------+----------+-------+---------------------------------+
Total jobs 1

patch> Job 100 completed
------------------- Final Summary --------------------
Number of databases            [ 1 ]

Jobs finished                  [1]
Jobs failed                    [0]
Jobs restored                  [0]
Jobs pending                   [0]

# Run the root.sh script as root for the following jobs:
For create_home_1 -> /u01/app/oracle/product/23.26.3/dbhome_1/root.sh

Please check the summary report at:
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/patch/auto/status/status.html
/u01/app/oracle/autoupgrade/logs/cfgtoollogs/patch/auto/status/status.log
[oracle@ms-vm-01 autoupgrade]$

```
## 4. Run Root.sh
```bash
[root@ms-vm-01 install]# /u01/app/oracle/product/23.26.3/dbhome_1/root.sh
Check /u01/app/oracle/product/23.26.3/dbhome_1/install/root_ms-vm-01.localdomain_2026-08-19_14-05-48-410011221.log for the output of root script
[root@ms-vm-01 install]#

[root@ms-vm-01 install]# cat /u01/app/oracle/product/23.26.3/dbhome_1/install/root_ms-vm-01.localdomain_2026-08-19_14-05-48-410011221.log
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER= oracle
    ORACLE_HOME=  /u01/app/oracle/product/23.26.3/dbhome_1
   Copying dbhome to /usr/local/bin ...
   Copying oraenv to /usr/local/bin ...
   Copying coraenv to /usr/local/bin ...

Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
[root@ms-vm-01 install]#
```


## 5. Validate Patch
```bash
[oracle@ms-vm-01 OPatch]$ export ORACLE_HOME=/u01/app/oracle/product/23.26.3/dbhome_1
[oracle@ms-vm-01 OPatch]$ export PATH=$ORACLE_HOME/OPatch:$PATH
[oracle@ms-vm-01 OPatch]$ ./opatch lspatches
39788260;Fix for Bug 39788260
39779540;39779540:Fix for Bug 39779540
39779336;Fix for Bug 39779336
39739695;Fix for Bug 39739695
39661089;Fix for Bug 39661089
39593097;DATAPUMP BUNDLE PATCH 23.26.3.0.0
39578859;OCW RELEASE UPDATE 23.26.3.0.0 (39578859) Gold Image
39578879;Database Release Update : 23.26.3.0.0 (39578879) Gold Image

OPatch succeeded.
[oracle@ms-vm-01 OPatch]$
```
