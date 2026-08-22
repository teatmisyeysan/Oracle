# Transfer AWR from one PDB 19c to another PDB in Oracle 19c
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
```bash
Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 2
Begin Snapshot Id specified: 2

Enter value for end_snap: 16
End   Snapshot Id specified: 16
```

## 5. List the Directory present in Database, Choose the directory location and dump file name "DATA_PUMP_DIR":
```bash
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
```
## 6. Copy dump from source to target server
```bash
[oracle@NR-UAT-SV01 57310CB8D27621F4E063F7C8760A29C3]$ scp awrdat_2_16_20260822.dmp awrdat_2_16_20260822__sm.dmp oracle@10.xxx.xxx.201:/u01/app/oracle/admin/cdb19c/dpdump/52A1101D1A195655E063C9C8760A7AAB
oracle@10.118.200.201's password:
awrdat_2_16_20260822.dmp                                                                                              100%   22MB 154.7MB/s   00:00
awrdat_2_16_20260822__sm.dmp                                                                                          100%  232KB 120.0MB/s   00:00
[oracle@NR-UAT-SV01 57310CB8D27621F4E063F7C8760A29C3]$
```
## 7. Validate dump on target
```bash
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$ ls -ltr awrdat_2_16*
-rw-r-----. 1 oracle oinstall 22982656 Aug 22 08:58 awrdat_2_16_20260822.dmp
-rw-r-----. 1 oracle oinstall   237568 Aug 22 08:58 awrdat_2_16_20260822__sm.dmp
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$ pwd
/u01/app/oracle/admin/cdb19c/dpdump/52A1101D1A195655E063C9C8760A7AAB
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$
```
## 8. Loading AWR Data
```bash
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$ echo $ORACLE_SID
cdb19c
[oracle@dev-dbserver01 52A1101D1A195655E063C9C8760A7AAB]$ sqlplus / as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Sat Aug 22 09:07:09 2026
Version 19.32.0.0.0

Copyright (c) 1982, 2026, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.32.0.0.0

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB19C                         READ WRITE NO
SQL>
SQL> alter session set container=pdb19c;

Session altered.

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 PDB19C                         READ WRITE NO
SQL>
SQL> @?/rdbms/admin/awrload.sql
~~~~~~~~~~
AWR LOAD
~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~  This script will load the AWR data from a dump file. The   ~
~  script will prompt users for the following information:    ~
~     (1) name of directory object                            ~
~     (2) name of dump file                                   ~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Specify the Directory Name
~~~~~~~~~~~~~~~~~~~~~~~~~~

Directory Name                 Directory Path
------------------------------ -------------------------------------------------
DATA_PUMP_DIR                  /u01/app/oracle/admin/cdb19c/dpdump/52A1101D1A195
                               655E063C9C8760A7AAB

DBMS_OPTIM_ADMINDIR            /u02/app/oracle/product/19.32/dbhome_1/rdbms/admi
                               n

DBMS_OPTIM_LOGDIR              /u02/app/oracle/product/19.32/dbhome_1/cfgtoollog
                               s

JAVA$JOX$CUJS$DIRECTORY$       /u02/app/oracle/product/19.32/dbhome_1/javavm/adm
                               in/

Directory Name                 Directory Path
------------------------------ -------------------------------------------------

OPATCH_INST_DIR                /u02/app/oracle/product/19.32/dbhome_1/OPatch
OPATCH_LOG_DIR                 /u02/app/oracle/product/19.32/dbhome_1/rdbms/log
OPATCH_SCRIPT_DIR              /u02/app/oracle/product/19.32/dbhome_1/QOpatch
ORACLE_BASE                    /u01/app/oracle
ORACLE_HOME                    /u02/app/oracle/product/19.32/dbhome_1
SDO_DIR_ADMIN                  /u02/app/oracle/product/19.32/dbhome_1/md/admin
SDO_DIR_WORK
SS_OE_XMLDIR                   /home/oracle/scripts/DBA/sample_schema/db-sample-
                               schemas-23.3/order_entry/order_entry/


Directory Name                 Directory Path
------------------------------ -------------------------------------------------
SUBDIR                         /home/oracle/scripts/DBA/sample_schema/db-sample-
                               schemas-23.3/order_entry/order_entry//2002/Sep

XMLDIR                         /u02/app/oracle/product/19.32/dbhome_1/rdbms/xml
XSDDIR                         /u02/app/oracle/product/19.32/dbhome_1/rdbms/xml/
                               schema

```
Specify the Directory Name
```bash
Choose a Directory Name from the list above (case-sensitive).
Enter value for directory_name:
```
```bash
Choose a Directory Name from the list above (case-sensitive).

Enter value for directory_name: DATA_PUMP_DIR

Using the dump directory: DATA_PUMP_DIR
```
Specify file name "awrdat_2_16_20260822"
```bash
Specify the Name of the Dump File to Load
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Please specify the prefix of the dump file (.dmp) to load:

Enter value for file_name: awrdat_2_16_20260822

Loading from the file name: awrdat_2_16_20260822.dmp


|
| ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
|  Loading the AWR data from the following
|  directory/file:
|   /u01/app/oracle/admin/cdb19c/dpdump/52A1101D1A195655E063C9C8760A7AAB
|   awrdat_2_16_20260822.dmp
| ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
|
|  *** AWR Load Started ...
|
|  This operation will take a few moments. The
|  progress of the AWR load operation can be
|  monitored in the following directory/file:
|   /u01/app/oracle/admin/cdb19c/dpdump/52A1101D1A195655E063C9C8760A7AAB
|   awrdat_2_16_20260822.log
|

End of AWR Load
SQL>

```

## 9. Generate AWR on target after loaded
```bash
[oracle@dev-dbserver01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Sat Aug 22 09:20:07 2026
Version 19.32.0.0.0

Copyright (c) 1982, 2026, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.32.0.0.0

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB19C                         READ WRITE NO
SQL>
SQL> alter session set container=pdb19c;

Session altered.

SQL> show pdbs;

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 PDB19C                         READ WRITE NO
SQL>
SQL>
SQL> @?/rdbms/admin/awrrpti.sql;

Specify the Report Type
~~~~~~~~~~~~~~~~~~~~~~~
AWR reports can be generated in the following formats.  Please enter the
name of the format at the prompt. Default value is 'html'.

   'html'          HTML format (default)
   'text'          Text format
   'active-html'   Includes Performance Hub active report

Enter value for report_type:



Type Specified: html


Specify the location of AWR Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
AWR_ROOT - Use AWR data from root (default)
AWR_PDB - Use AWR data from PDB
Enter value for awr_location: AWR_PDB

Location of AWR Data Specified: AWR_PDB


Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  DB Id      Inst Num   DB Name      Instance     Host
------------ ---------- ---------    ----------   ------
  228667876      1      CDB19C       cdb19c       NR-UAT-SV01
  2332625880     1      CDB19C       cdb19c       dev-dbserver

Enter value for dbid: 228667876
Using 228667876 for database Id
Enter value for inst_num: 1
Using 1 for instance number


Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.


Enter value for num_days: 3

Listing the last 3 days of Completed Snapshots
Instance     DB Name      Snap Id       Snap Started    Snap Level
------------ ------------ ---------- ------------------ ----------

cdb19c       CDB19C               1  21 Aug 2026 16:46    1
                                  2  21 Aug 2026 18:00    1
                                  3  21 Aug 2026 19:00    1
                                  4  21 Aug 2026 20:00    1
                                  5  21 Aug 2026 21:00    1
                                  6  21 Aug 2026 22:00    1
                                  7  21 Aug 2026 23:00    1
                                  8  22 Aug 2026 00:00    1
                                  9  22 Aug 2026 01:00    1
                                 10  22 Aug 2026 02:00    1
                                 11  22 Aug 2026 03:00    1
                                 12  22 Aug 2026 04:00    1
                                 13  22 Aug 2026 05:00    1
                                 14  22 Aug 2026 06:00    1
                                 15  22 Aug 2026 07:00    1
                                 16  22 Aug 2026 08:00    1


Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 15
Begin Snapshot Id specified: 15

Enter value for end_snap: 16
End   Snapshot Id specified: 16



Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrrpt_1_15_16.html.  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: awrrpt_1_15_16_pdb19c.html

Using the report name awrrpt_1_15_16_pdb19c.html

<html lang="en"><head><title>AWR Report for DB: CDB19C, Inst: cdb19c, Snaps: 15-16</title>
........

<p />
End of Report
</body></html>
Report written to awrrpt_1_15_16_pdb19c.html
SQL>
```
<img width="950" height="696" alt="image" src="https://github.com/user-attachments/assets/4a7fe072-d6b2-4758-b931-ff0ff45abb35" />



