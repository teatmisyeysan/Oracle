## Bug 38622857
https://docs.oracle.com/en/database/oracle/oracle-database/26/rnrdm/bug-38622857.html
```bash
[grid@ms-vm-01 grid]$ ./gridSetup.sh
ERROR: Unable to verify the graphical display setup. This application requires X display. Make sure that xdpyinfo exist under PATH variable.
Launching Oracle Grid Infrastructure Setup Wizard...

The response file for this session can be found at:
 /u01/app/26.0.0/grid/install/response/grid_2026-08-27_10-02-53PM.rsp

You can find the log of this install session at:
 /tmp/GridSetupActions2026-08-27_10-02-53PM/gridSetupActions2026-08-27_10-02-53PM.log
Moved the install session logs to:
 /u01/app/oraInventory/logs/GridSetupActions2026-08-27_10-02-53PM
[grid@ms-vm-01 grid]$

```

<img width="1137" height="202" alt="image" src="https://github.com/user-attachments/assets/4fd2aa54-9a5e-47d1-a377-361c99de1c20" />


# 1
<img width="806" height="635" alt="image" src="https://github.com/user-attachments/assets/5320528f-828f-4464-99d4-7ed7ee555476" />

# 2
<img width="797" height="632" alt="image" src="https://github.com/user-attachments/assets/cf79e218-e968-4d05-8feb-ce9f44cc9d13" />

# 3
<img width="797" height="632" alt="image" src="https://github.com/user-attachments/assets/cafb6c93-f6df-44d8-927b-7686e91ecb56" />

# 4
<img width="801" height="632" alt="image" src="https://github.com/user-attachments/assets/8b7274c9-60e2-4d63-9c65-9c0166905e9d" />

# 5
<img width="797" height="636" alt="image" src="https://github.com/user-attachments/assets/b07cb1ee-c7c4-4f8b-a9ae-2fb943188f87" />

# 6
<img width="802" height="632" alt="image" src="https://github.com/user-attachments/assets/e7d0a548-1a60-4f10-8359-45e4b7bfc166" />

# 7
<img width="802" height="630" alt="image" src="https://github.com/user-attachments/assets/f0abeace-024d-460a-940c-b56e0c8070c4" />

# 8
<img width="802" height="632" alt="image" src="https://github.com/user-attachments/assets/567050c5-65af-4b8a-9602-8928f8584629" />

# 9
<img width="797" height="631" alt="image" src="https://github.com/user-attachments/assets/f3de60f3-b532-4ae1-8e74-67d6baa82d48" />

# 10
<img width="800" height="630" alt="image" src="https://github.com/user-attachments/assets/e7a2a13a-516d-4ebc-924e-5c193e7bc502" />

# 11
<img width="802" height="636" alt="image" src="https://github.com/user-attachments/assets/2d9a6b7c-eaf4-4b55-b3c7-634d81d49189" />

# 12
<img width="800" height="636" alt="image" src="https://github.com/user-attachments/assets/6627b19c-1129-48c8-8bb4-a7633e20036f" />

# 13
<img width="805" height="635" alt="image" src="https://github.com/user-attachments/assets/7eb86f9c-fa09-4b3a-abbb-f96f1d8b636f" />

# 14
```bash
[root@ms-vm-01 ~]# /u01/app/oraInventory/orainstRoot.sh
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/app/oraInventory to oinstall.
The execution of the script is complete.
[root@ms-vm-01 ~]#
[root@ms-vm-01 ~]# /u01/app/26.0.0/grid/root.sh
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER= grid
    ORACLE_HOME=  /u01/app/26.0.0/grid

Enter the full pathname of the local bin directory: [/usr/local/bin]:
   Copying dbhome to /usr/local/bin ...
   Copying oraenv to /usr/local/bin ...
   Copying coraenv to /usr/local/bin ...


Creating /etc/oratab file...
Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
Executing command '/u01/app/26.0.0/grid/perl/bin/perl -I/u01/app/26.0.0/grid/perl/lib -I/u01/app/26.0.0/grid/crs/install /u01/app/26.0.0/grid/crs/install/roothas.pl '
Using configuration parameter file: /u01/app/26.0.0/grid/crs/install/crsconfig_params
The log of current session can be found at:
  /u01/app/grid/crsdata/ms-vm-01/crsconfig/roothas_2026-08-27_10-16-19PM.log
Redirecting to /bin/systemctl restart rsyslog.service
LOCAL ADD MODE
Creating OCR keys for user 'grid', privgrp 'oinstall'..
Operation successful.
2026/08/27 22:16:52 CLSRSC-330: Adding Clusterware entries to file 'oracle-ohasd.service'

ms-vm-01     2026/08/27 22:17:29     /u01/app/grid/crsdata/ms-vm-01/olr/backup_20260827_221729.olr     2107015493
2026/08/27 22:17:30 CLSRSC-327: Successfully configured Oracle Restart for a standalone server
[root@ms-vm-01 ~]#

```

<img width="1357" height="605" alt="image" src="https://github.com/user-attachments/assets/ff4344f1-f5a2-400e-b8c7-f139c39d17f8" />

# 15

<img width="802" height="635" alt="image" src="https://github.com/user-attachments/assets/f93d5595-5d10-4140-a620-c7914e3dccd2" />

# 16
```bash
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$ ps -ef |grep pmon
grid       83088       1  0 22:19 ?        00:00:00 asm_pmon_+ASM
grid       84856   57357  0 22:22 pts/7    00:00:00 grep --color=auto pmon
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$ sqlplus / as sysasm

SQL*Plus: Release 23.26.1.0.0 - Production on Thu Aug 27 22:22:40 2026
Version 23.26.1.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.


Connected to:
Oracle AI Database 26ai Enterprise Edition Release 23.26.1.0.0 - Production
Version 23.26.1.0.0

SQL>
SQL>
SQL> exit
Disconnected from Oracle AI Database 26ai Enterprise Edition Release 23.26.1.0.0 - Production
Version 23.26.1.0.0
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$ asmcmd lsdg
State    Type    Rebal  Sector  Logical_Sector  Block       AU  Total_MB  Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  EXTERN  N         512             512   4096  4194304     20476    20372                0           20372              0             N  DATA/
[grid@ms-vm-01 grid]$
[grid@ms-vm-01 grid]$

```

<img width="1225" height="467" alt="image" src="https://github.com/user-attachments/assets/610aa468-66d1-43dc-bfa2-6ff100c7a35b" />


# RDBMS Setup
# 1
```bash
[oracle@ms-vm-01 dbhome_1]$
[oracle@ms-vm-01 dbhome_1]$ ./runInstaller
ERROR: Unable to verify the graphical display setup. This application requires X display. Make sure that xdpyinfo exist under PATH variable.
Launching Oracle AI Database Setup Wizard...
```

<img width="805" height="632" alt="image" src="https://github.com/user-attachments/assets/917ebe8b-1106-4bdc-b581-c2543264630e" />

# 2

<img width="801" height="632" alt="image" src="https://github.com/user-attachments/assets/e27b49a7-9646-4f6a-bfea-21d6828b9390" />

# 3

<img width="797" height="632" alt="image" src="https://github.com/user-attachments/assets/1047b9c4-f2aa-4d4a-b836-60b6d6738f41" />

# 4

<img width="802" height="632" alt="image" src="https://github.com/user-attachments/assets/6e6351e8-e145-4e0c-ba37-be65f7e975d4" />

# 5

<img width="800" height="635" alt="image" src="https://github.com/user-attachments/assets/bc0ea0e2-4436-4de8-b426-9e5afe0bfb9f" />

# 6

<img width="796" height="632" alt="image" src="https://github.com/user-attachments/assets/b4bd4c07-c067-4f1c-b793-bb50cb6b8d1f" />

# 7

<img width="802" height="630" alt="image" src="https://github.com/user-attachments/assets/7e8f843d-56ab-47b7-b7cf-a1a4b7f642be" />

# 8

<img width="806" height="635" alt="image" src="https://github.com/user-attachments/assets/d6b9cc6d-cdc3-4be4-8cad-1701f488955c" />

# 9

<img width="797" height="631" alt="image" src="https://github.com/user-attachments/assets/ccec2eea-aec5-4d2a-8610-88b8964d5a7e" />

# 10
```bash
[root@ms-vm-01 ~]#
[root@ms-vm-01 ~]# /u01/app/oracle/product/26.0.0/dbhome_1/root.sh
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER= oracle
    ORACLE_HOME=  /u01/app/oracle/product/26.0.0/dbhome_1

Enter the full pathname of the local bin directory: [/usr/local/bin]:
The contents of "dbhome" have not changed. No need to overwrite.
The contents of "oraenv" have not changed. No need to overwrite.
The contents of "coraenv" have not changed. No need to overwrite.

Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
[root@ms-vm-01 ~]#

```

<img width="597" height="257" alt="image" src="https://github.com/user-attachments/assets/ee660018-4ca5-4d37-b527-9119a27048cc" />


# 11

<img width="802" height="632" alt="image" src="https://github.com/user-attachments/assets/b816ebe6-b536-435c-86c5-05dcbfd190ca" />

# DBCA 
# 1

<img width="802" height="636" alt="image" src="https://github.com/user-attachments/assets/2a926425-60c5-45f9-8ed0-126c33cf3d4d" />

# 2

<img width="800" height="632" alt="image" src="https://github.com/user-attachments/assets/1cd10fc3-43f3-468d-afba-8317e209fc0c" />

# 3

<img width="801" height="632" alt="image" src="https://github.com/user-attachments/assets/654645be-4930-49b8-8223-ca0ee100fd85" />

# 4

<img width="800" height="635" alt="image" src="https://github.com/user-attachments/assets/f3f1dac1-b8b8-4a7a-b807-91c65b88d520" />

# 5

<img width="515" height="287" alt="image" src="https://github.com/user-attachments/assets/19637dd4-1553-4841-b374-4a085401fdc1" />


<img width="800" height="632" alt="image" src="https://github.com/user-attachments/assets/37ebfc8e-8caa-47f2-9f7d-f5b5da363ac7" />

# 6

<img width="800" height="637" alt="image" src="https://github.com/user-attachments/assets/a3d2167b-791e-434b-ae6a-513610a4f1d4" />

# 7

<img width="800" height="635" alt="image" src="https://github.com/user-attachments/assets/7981ad57-5889-4e92-8da8-d0d08d9bff81" />

# 8

<img width="797" height="632" alt="image" src="https://github.com/user-attachments/assets/6b290699-e55c-4cd1-8595-7c07e8c323a5" />

# 9

<img width="797" height="632" alt="image" src="https://github.com/user-attachments/assets/daf0f164-faef-4cfc-88f4-e47fd28782ae" />

# 10

<img width="800" height="632" alt="image" src="https://github.com/user-attachments/assets/79341c93-49b4-46c8-8bf8-8396e968ca04" />

# 11

<img width="800" height="632" alt="image" src="https://github.com/user-attachments/assets/f7a2cfa2-c90d-4009-895b-b811c33e862c" />

# 12

<img width="802" height="636" alt="image" src="https://github.com/user-attachments/assets/f6550f58-4383-4836-93e6-acf2c60efd6c" />

# 13

<img width="801" height="631" alt="image" src="https://github.com/user-attachments/assets/b1733a84-1865-4c73-845e-95521f08dbdf" />

# 14

<img width="801" height="637" alt="image" src="https://github.com/user-attachments/assets/764f32c5-c9a8-4c45-93a4-7894f0d4d2e3" />

# 15

<img width="802" height="635" alt="image" src="https://github.com/user-attachments/assets/22e4d261-b059-42f8-b72a-05f400698245" />

# 16
