
# Install a New Oracle home for Oracle AI Database 26ai
* Prerequisites

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
```
--------------------------------------------------------

[oracle@ms-vm-01 autoupgrade]$
```
