# Multitenant 19c New Features

## Introduction
In this lab you will try out new features of Multitenant. You will create and configure a PDB snapshot carousel which is a library of point-in-time copy of a PDB. You will set up restore points specific for a pluggable database (PDB) to flashback to that restore point without affecting other PDBs in a multitenant container database (CDB). You will also create the configuration to monitor and manage a fleet of container databases (CDBs) and hosted pluggable databases (PDBs).

Estimated Time: 2 hour

[](youtube:kzTQGs75IjA)


### Prerequisites

This feature is currently restricted to Enterprise Edition on Engineered Systems, like Exadata, and Enterprise Edition on Oracle Database Cloud Services, as described [here](https://docs.oracle.com/en/database/oracle/oracle-database/19/dblic/Licensing-Information.html#GUID-0F9EB85D-4610-4EDF-89C2-4916A0E7AC87). There is a workaround for testing by enabling the "_exadata_feature_on" initialisation parameter.

**NOTE:** *When doing Copy/Paste using the convenient* **Copy** *function used throughout the guide, you must hit the* **ENTER** *key after pasting. Otherwise the last line will remain in the buffer until you hit* **ENTER!**

```
<copy>
export ORACLE_SID=CDB1
export ORAENV_ASK=NO
. oraenv
export ORAENV_ASK=YES

sqlplus / as sysdba <<EOF

alter system set "_exadata_feature_on"=true scope=spfile;
shutdown immediate;
startup;

exit;
EOF
</copy>
```

## Task 1: Log in and create PDB Snapshot Carousel

Support for PDB snapshots can be defined during PDB creation as part of the **CREATE PLUGGABLE DATABASE** statement, or after PDB creation using the **ALTER PLUGGABLE DATABASE** statement. The **SNAPSHOT MODE** clause accepts one of the following settings.

* NONE : The PDB does not support snapshots.
* MANUAL : The PDB supports snapshots, but they are only created manually requested.
* EVERY n HOURS : A snapshot is automatically created every "n" hours. Where "n" is between 1 and 1999.
* EVERY n MINUTES : A snapshot is automatically created every "n" minutes. Where "n" is between 1 and 2999.

**NOTE:** If you are using Oracle Managed Files (OMF) you don't need to worry about file name conversion. If you aren't using OMF, all PDB creation statements will require file name conversion using the **FILE_NAME_CONVERT** or **CREATE_FILE_DEST** settings in the CREATE PLUGGABLE DATABASE statements.

1. All scripts for this lab are stored in the `labs/multitenant` folder and are run as the oracle user. Let's navigate to the path now.

    ```
    <copy>
    cd /home/oracle/labs/multitenant
    </copy>
    ```

2.  Set your oracle environment and connect to **CDB1** using SQLcl.

    ```
    <copy>. ~/.set-env-db.sh CDB1</copy>
    ```

    ```
    <copy>
    sql sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    </copy>
    ```

    To make the SQLcl output easier to read on the screen, set the sql format.

    ```
    <copy>
    set sqlformat ANSICONSOLE
    </copy>
    ```

    ![](./images/task1.2-connectcdb1.png " ")

3. Create a script to check to see who you are connected as. At any point in the lab you can run this script to see who or where you are connected. We'll save this script for future use and call it "whoami.sql".

    ```
    <copy>
    select
      'DB Name: '  ||Sys_Context('Userenv', 'DB_Name')||
      ' / CDB?: '     ||case
        when Sys_Context('Userenv', 'CDB_Name') is not null then 'YES'
          else  'NO'
          end||
      ' / Auth-ID: '   ||Sys_Context('Userenv', 'Authenticated_Identity')||
      ' / Sessn-User: '||Sys_Context('Userenv', 'Session_User')||
      ' / Container: ' ||Nvl(Sys_Context('Userenv', 'Con_Name'), 'n/a')
      "Who am I?"
      from Dual
      .

      save whoami.sql
      /

    </copy>
    ```

    Now, let's run the script.

    ```
    <copy>
     @whoami.sql
    </copy>
    ```

   ![](./images/task1.3-whoisconnected.png " ")


4. Create a PDB Snapshot Carousel **PDB_SNAP2**.

    ```
    <copy>show pdbs
    </copy>
    ```

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba

    create pluggable database PDB_SNAP2 admin user PDB_Admin identified by Ora_DB4U snapshot mode every 1 hours;

    alter pluggable database PDB_SNAP2 open;

    show pdbs
    </copy>
    ```

    ![](./images/task1.4-createpdb2.png " ")

5. Create a script to check to display PDB snapshot carousel settings. The snapshot settings are displayed using the SNAPSHOT_MODE and SNAPSHOT_INTERVAL columns in the CDB_PDBS view. Notice the SNAPSHOT_INTERVAL values are displayed in minutes. We'll save this script for future use and call it "pdb_snapshot_mode.sql".

    ```
    <copy>
      COLUMN pdb_name FORMAT A10
      COLUMN snapshot_mode FORMAT A15
  
      SELECT p.con_id,
          p.pdb_name,
          p.snapshot_mode,
          p.snapshot_interval
      FROM   cdb_pdbs p
      ORDER BY 1
      .

      save pdb_snapshot_mode.sql
      /

    </copy>
    ```

    Now, let's run the script.

    ```
    <copy>
     @pdb_snapshot_mode.sql
    </copy>
    ```


6. Change the session to point to **PDB_SNAP2**.

    ```
    <copy>alter session set container = PDB_SNAP2;</copy>
    ```

    ![](./images/task1.5-altertopdb2.png " ")

7. Alter the snapshot settings using the **ALTER PLUGGABLE DATABASE** command in **PDB_SNAP2**.

    ```
    <copy>
    alter pluggable database snapshot mode every 1 minutes;

    </copy>
    ```

    ```
    <copy>
    alter pluggable database snapshot mode manual;

    </copy>
    ```
    ```
    <copy>
    alter pluggable database snapshot mode none;

    </copy>
    ````

    ```
    <copy>
    alter pluggable database snapshot mode every 30 minutes;

    </copy>
    ````

   ![](./images/task1.6-grantpdbadminprivs.png " ")


8. Create a script to report current MAX_PDB_SNAPSHOT parameter value using CDB_PROPERTIES view in **PDB_SNAP2**, save the script as max_pdb_snapshots.sql.

    ```
    <copy>
        SET LINESIZE 150 TAB OFF
        COLUMN property_name FORMAT A20
        COLUMN pdb_name FORMAT A10
        COLUMN property_value FORMAT A15
        COLUMN description FORMAT A50

        SELECT pr.con_id,
            p.pdb_name,
            pr.property_name, 
            pr.property_value,
            pr.description 
        FROM   cdb_properties pr
            JOIN cdb_pdbs p ON pr.con_id = p.con_id 
        WHERE  pr.property_name = 'MAX_PDB_SNAPSHOTS' 
        ORDER BY pr.property_name
      .

      save max_pdb_snapshots.sql
      /

    </copy>
    ```

    Now, let's run the script.

    ```
    <copy>
     @max_pdb_snapshots.sql
    </copy>
    ```

    ```
    <copy>
    alter pluggable database set max_pdb_snapshots=8;
    </copy>
    ```

   ![](./images/task1.8-createtable_pdbadmin.png " ")

9. Take snapshot of the PDB manually 

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    alter session set container = PDB_SNAP2;
    </copy>
    ```
    
    ```
    <copy>
    alter pluggable database snapshot;
    </copy>
    ```

    ```
    <copy>    
    alter pluggable database snapshot my_snapshot;
    </copy>
    ```

    ```
    <copy>    
    alter pluggable database snapshot my_snapshot_save;
    </copy>
    ```

10. List available snapshots of PDB using **CDB_PDB_SNAPSHOTS** view. Save the script as pdb_snapshots.sql.

    ```
    <copy>
        SET LINESIZE 150 TAB OFF
        COLUMN con_name FORMAT A10
        COLUMN snapshot_name FORMAT A30
        COLUMN snapshot_scn FORMAT 9999999
        COLUMN full_snapshot_path FORMAT A50

        SELECT con_id,
            con_name,
            snapshot_name, 
            snapshot_scn,
            full_snapshot_path 
        FROM   cdb_pdb_snapshots
        ORDER BY con_id, snapshot_scn
      .

      save pdb_snapshots.sql
      /

    </copy>
    ```

    Now, let's run the script.

    ```
    <copy>
     @pdb_snapshots.sql
    </copy>
    ```

11. Drop the snapshot manually. It works for snapshots with user-defined or system generated names.

    ```
    <copy>
    alter session set container = PDB_SNAP2;
    </copy>
    ```
    
    ```
    <copy>
    alter pluggable database drop snapshot my_snapshot;
    </copy>
    ```

    ```
    <copy>    
    @pdb_snapshots.sql
    </copy>
    ```

12. Create PDB clone from PDB snapshot.

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    </copy>
    ```
    
    ```
    <copy>
    create pluggable database pdb2_my_snapshot from pdb_snap2 using snapshot my_snapshot_save snapshot mode every 24 hours;
    </copy>
    ```

    ```
    <copy>
    alter pluggable database pdb2_my_snapshot open;
    </copy>
    ```

    ```
    <copy>    
    @pdb_snapshot_mode.sql
    </copy>
    ```

    ```
    <copy>    
    exit
    </copy>
    ```

## Task 2: PDB Flashback
In Oracle Database 12.1 flashback database operations were limited to the root container, and therefore affected all pluggable databases (PDBs) associated with the root container. Oracle Database now supports flashback of a pluggable database, making flashback database relevant in the multitenant architecture again.

The task you will do in this step is:
- Create Restore Points in **PDB2** and Flashback database **PDB2** to restore point to rollback unwanted changes. 

    If you're not already running SQLcl, then launch SQLcl and set the formatting to make the on-screen output easier to read.

    ```
    <copy>sql /nolog
    set sqlformat ANSICONSOLE
    </copy>
    ```

1. Enable/Disable Flashback Database **CDB1**.

    ```
    <copy>
    sql /nolog
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    set sqlformat ANSICONSOLE
    </copy>
    ```

    ```
    <copy>
    shutdown immediate
    startup mount
    alter database archivelog;
    alter database open;
    </copy>
    ```

    ```
    <copy>
    alter database flashback on;
    select name, open_mode, flashback_on from v$database;
    </copy>
    ```
2. Update database parameter **DB_FLASHBACK_RETENTION_TARGET** to retain amount of flashback logs for 7 days.

    ```
    <copy>
    alter system set db_flashback_retention_target=10080 scope=both;
    </copy>
    ```

   ![](./images/task2.2-alterpdbreadonly.png " ")


3. Create Restore Points at PDB level in **PDB2**.

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba

    alter session set container=PDB2;
    </copy>
    ```
    
    ```
    <copy>
    create restore point PDB2_BEFORE_T1;
    </copy>
    ```

   ![](./images/task2.3-clonepdb2topdb3.png " ")

4. Create table t1 in **PDB2**.

    ```
    <copy>
    create table T1 (a number, b varchar2(50)); 
    insert into T1 values (1, 'A'); 
    commit;
    select * from T1;
    </copy>
    ```

   ![](./images/task2.4-pdb2readwrite.png " ")

5. Create Guaranteed Restore Point in root container for **PDB2**.

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    show pdbs
    </copy>
    ```

    ```
    <copy>
    create restore point PDB2_BEFORE_T2 for pluggable database PDB2 guarantee flashback database;
    </copy>
    ```

    ```
    <copy>
    select name, scn, guarantee_flashback_database from v$restore_point; 
    </copy>
    ```

   ![](./images/task2.3-clonepdb2topdb3.png " ")

6. Create table t2 in **PDB2**

    ```
    <copy>
    alter session set container=PDB2;
    </copy>
    ```
   
    ```
    <copy>
    create table T2 (x number); 
    insert into T2 values (100); 
    commit;
    select * from T2;
    </copy>
    ```

    ```
    <copy>
    select table_name from user_tables where table_name in ('T1','T2');
    </copy>
    ```   

7. Flashback database **PDB2** to GRP **PDB2_BEFORE_T2** 

    ```
    <copy>
    connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba
    </copy>
    ```

    ```
    <copy>
    alter pluggable database PDB2 close;
    flashback pluggable database PDB2 to restore point PDB2_BEFORE_T2;
    alter pluggable database PDB2 open resetlogs;
    </copy>
    ```

    ```
    <copy>
    select table_name from user_tables where table_name in ('T1','T2');
    select * from T2;
    </copy>
    ``` 
## Task 3: CDB Fleet Management
This section reviews how to unplug a PDB from its container database (CDB) and save the PDB metadata into an XML manifest file.

The task you will do in this step is:
- Unplug **PDB3** from **CDB1**

    If you are not already running SQLcl, then launch SQLcl and set the formatting to make the on-screen output easier to read.

    ```
    <copy>sql /nolog
    set sqlformat ANSICONSOLE
    </copy>
    ```

1. Connect to the container **CDB1**.

    ```
    <copy>connect sys/Ora_DB4U@localhost:1521/cdb1 as sysdba</copy>
    ```

2. Unplug **PDB3** from **CDB1**.

    ```
    <copy>show pdbs</copy>
    ```

    ```
    <copy>alter pluggable database PDB3 close immediate;</copy>
    ```

    ```
    <copy>
    alter pluggable database PDB3
    unplug into
    '/opt/oracle/oradata/CDB1/pdb3.xml';
    show pdbs
    </copy>
    ```


   ![](./images/task3.2-unplugpdb3.png " ")

3. Remove **PDB3** from **CDB1** but keep the PDB datafiles for future use.

    ```
    <copy>drop pluggable database PDB3 keep datafiles;
    show pdbs
    </copy>
    ```

   ![](./images/task3.3-droppdb3.png " ")

4. Show the datafiles in **CDB1** and note that files for PDB3 are no longer part of the container.
    ```
    <copy>
    with Containers as (
      select PDB_ID Con_ID, PDB_Name Con_Name from DBA_PDBs
      union
      select 1 Con_ID, 'CDB$ROOT' Con_Name from Dual)
    select
      Con_ID,
      Con_Name "Con_Name",
      Tablespace_Name "T'space_Name",
      File_Name "File_Name"
    from CDB_Data_Files inner join Containers using (Con_ID)
    union
    select
      Con_ID,
      Con_Name "Con_Name",
      Tablespace_Name "T'space_Name",
      File_Name "File_Name"
    from CDB_Temp_Files inner join Containers using (Con_ID)
    order by 1, 3
    /
    </copy>
    ```

    ![](./images/task3.4-cdb1dbfiles.png " ")

5. Look at the XML file for the pluggable database **PDB3**.  Take some time to review the information that is contained in a PDB manifest.

    ```
    <copy>host cat /opt/oracle/oradata/CDB1/pdb3.xml</copy>
    ```

    ![](./images/task3.5-catxmlfile.png " ")

## Task 11: Lab cleanup

1. Exit from the SQL command prompt and reset the container databases back to their original ports. If any errors about dropping databases appear, you can ignore them.

    ```
    <copy>
    exit
    </copy>
    ```
    ```
    <copy>
    ./resetCDB.sh
    </copy>
    ```

    ![](./images/task11.1-labcleanup.png " ")

Now you've had a chance to try out the Multitenant option. You were able to create, clone, plug and unplug a pluggable database. You were then able to accomplish some advanced tasks, such as hot cloning and PDB relocation, that you could leverage to more easily maintain a large multitenant environment.

Please *proceed to the next lab*.

## Acknowledgements

- **Author** - Patrick Wheeler, VP, Multitenant Product Management
- **Contributors** -  Joseph Bernens, David Start, Anoosha Pilli, Brian McGraw, Quintin Hill, Rene Fontcha, Alfredo Krieg, Royce Fu
- **Last Updated By/Date** - Royce Fu, North America Cloud & Technology, October 2021