# Transfer AWR data from one system to another in Oracle
Extract AWR data
For Extract the AWR data from source database 19c run the awrextr.sql script which extracts AWR data for a range of snapshots from the database into a DataPump export file.

## 1. Run the following script for extract AWR:
```bash
[oracle@NR-UAT-SV01 ~]$ sql

SQL*Plus: Release 19.0.0.0.0 - Production on Sat Aug 22 08:27:16 2026
Version 19.31.0.0.0

Copyright (c) 1982, 2026, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.31.0.0.0

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB19C                         READ WRITE NO
SQL> alter session set container=pdb19c;

Session altered.

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 PDB19C                         READ WRITE NO
SQL> @?/rdbms/admin/awrextr.sql;
~~~~~~~~~~~~~
AWR EXTRACT
~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~  This script will extract the AWR data for a range of snapshots  ~
~  into a dump file.  The script will prompt users for the         ~
~  following information:                                          ~
~     (1) database id                                              ~
~     (2) snapshot range to extract                                ~
~     (3) name of directory object                                 ~
~     (4) name of dump file                                        ~
~     (5) export sql monitor data or not                           ~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Databases in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     DB Name      Host
------------ ------------ ------------
  228667876  CDB19C       NR-UAT-SV01

The default database id is the local one: '228667876'.  To use this
database id, press <return> to continue, otherwise enter an alternative.

```
## 2. Script ask for select DBID
```bash
Using 228667876 for Database ID
```
## 3. Enter the number of days backup export:
```bash
Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.



Listing the last 3 days of Completed Snapshots

DB Name        Snap Id    Snap Started
------------ --------- ------------------
CDB19C               1 21 Aug 2026 16:46
                     2 21 Aug 2026 18:00
                     3 21 Aug 2026 19:00
                     4 21 Aug 2026 20:00
                     5 21 Aug 2026 21:00
                     6 21 Aug 2026 22:00
                     7 21 Aug 2026 23:00
                     8 22 Aug 2026 00:00
                     9 22 Aug 2026 01:00
                    10 22 Aug 2026 02:00
                    11 22 Aug 2026 03:00
                    12 22 Aug 2026 04:00
                    13 22 Aug 2026 05:00
                    14 22 Aug 2026 06:00
                    15 22 Aug 2026 07:00
                    16 22 Aug 2026 08:00

```
## 4. It will list the 3 days snapshot in AWR. Choose the begin and end snapshot for export:
Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 2
Begin Snapshot Id specified: 2

Enter value for end_snap: 16
End   Snapshot Id specified: 16
```

## 5. List the Directory present in Database, Choose the directory location and dump file name "DATA_PUMP_DIR":
```basg
Specify the Directory Name
~~~~~~~~~~~~~~~~~~~~~~~~~~

Directory Name                 Directory Path
------------------------------ -------------------------------------------------
DATA_PUMP_DIR                  /u01/app/oracle/admin/cdb19c/dpdump/57310CB8D2762
                               1F4E063F7C8760A29C3
DBMS_OPTIM_ADMINDIR            /u01/app/oracle/product/19.31/dbhome_1/rdbms/admi
                               n
DBMS_OPTIM_LOGDIR              /u01/app/oracle/product/19.31/dbhome_1/cfgtoollog
                               s
JAVA$JOX$CUJS$DIRECTORY$       /u01/app/oracle/product/19.31/dbhome_1/javavm/adm
                               in/
OPATCH_INST_DIR                /u01/app/oracle/product/19.31/dbhome_1/OPatch
OPATCH_LOG_DIR                 /u01/app/oracle/product/19.31/dbhome_1/rdbms/log
OPATCH_SCRIPT_DIR              /u01/app/oracle/product/19.31/dbhome_1/QOpatch
ORACLE_BASE                    /u01/app/oracle
ORACLE_HOME                    /u01/app/oracle/product/19.31/dbhome_1
SDO_DIR_ADMIN                  /u01/app/oracle/product/19.31/dbhome_1/md/admin
SDO_DIR_WORK
XMLDIR                         /u01/app/oracle/product/19.31/dbhome_1/rdbms/xml
XSDDIR                         /u01/app/oracle/product/19.31/dbhome_1/rdbms/xml/
                               schema

Choose a Directory Name from the above list (case-sensitive).

Enter value for directory_name: DATA_PUMP_DIR

Using the dump directory: DATA_PUMP_DIR

Specify the Name of the Extract Dump File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The prefix for the default dump file name is awrdat_2_16.
To use this name, press <return> to continue, otherwise enter
an alternative.

Enter value for file_name: awrdat_2_16_20260822

Using the dump file prefix: awrdat_2_16_20260822

Specify whether to export sql monitor data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If not to export sql monitor data,
enter NO or press <return> to continue,
otherwise enter YES to export sql monitor data

Enter value for include_sqlmon: YES
|
| ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
|  The AWR extract dump file will be located
|  in the following directory/file:
|   /u01/app/oracle/admin/cdb19c/dpdump/57310CB8D27621F4E063F7C8760A29C3
|   awrdat_2_16_20260822.dmp
| ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
|
|  *** AWR Extract Started ...
|
|  This operation will take a few moments. The
|  progress of the AWR extract operation can be
|  monitored in the following directory/file:
|   /u01/app/oracle/admin/cdb19c/dpdump/57310CB8D27621F4E063F7C8760A29C3
|   awrdat_2_16_20260822.log
|

End of AWR Extract
SQL>



