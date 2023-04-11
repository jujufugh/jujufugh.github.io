# Sample Schema Setup

## Introduction
This lab will show you how to setup your database schemas for the subsequent labs.

Estimated Time: 10 minutes

### Objectives

In this lab, you will setup sample schema:
* Set the environment variables
* Get the Database sample schemas and unzip them
* Install the Sample Schemas

### Prerequisites
This lab assumes you have:

* A LiveLabs Cloud account and assigned compartment
* The IP address and instance name for your DB19c Compute instance
* Successfully logged into your LiveLabs account
* A Valid SSH Key Pair

## Task1: Install Sample Data

In this step, you will install a selection of the Oracle Database Sample Schemas.  For more information on these schemas, please review the Schema agreement at the end of this lab.

By completing the instructions below the sample schemas **SH**, **OE**, and **HR** will be installed. These schemas are used in Oracle documentation to show SQL language concepts and other database features. The schemas themselves are documented in Oracle Database Sample Schemas [Oracle Database Sample Schemas](https://www.oracle.com/pls/topic/lookup?ctx=dblatest&id=COMSC).

1. Run a *whoami* to ensure the value *oracle* comes back.)

    Note: If you are running in Windows using putty, ensure your Session Timeout is set to greater than 0.
    ```
    <copy>whoami</copy>
    ```

2. If you are not the oracle user, log back in:
    ````
    <copy>
    sudo su - oracle
    </copy>
    ````

    ![substitute user oracle](./images/sudo-oracle.png " ")

3.  Set the environment variables to point to the Oracle binaries.  When prompted for the SID (Oracle Database System Identifier), enter **cdb1**.
    ````
    <copy>
    . oraenv
    </copy>
    cdb1
    ````
    ![environment variables](./images/oraenv.png " ")

4. Get the Database sample schemas and unzip them. Then set the path in the scripts.

    ````
    <copy>
    wget https://github.com/oracle/db-sample-schemas/archive/v19c.zip
    unzip v19c.zip
    cd db-sample-schemas-19c
    perl -p -i.bak -e 's#__SUB__CWD__#'$(pwd)'#g' *.sql */*.sql */*.dat
    </copy>
    ````

    ![install schema](./images/install-schema-zip1.png " " )
    ![install schema](./images/install-schema-zip2.png " " )

5.  Login using SQL*Plus as the **oracle** user and create TEST_DATA tablespace to prepare for sample schemas installation.  

    ````
    <copy>
    sqlplus system/Oracle123@localhost:1521/pdb1
    create tablespace TEST_DATA datafile '/u01/oradata/cdb1/pdb1/test_data.dbf' size 300m autoextend on;
    </copy>
    ````
    ![start sqlplus](./images/start-sqlplus.png " ")

6.  Install the Sample Schemas by running the script below.

    ````
    <copy>
    @/home/oracle/DBSecLab/db-sample-schemas-19c/sales_history/sh_main Oracle123 test_data temp Oracle123 /home/oracle/DBSecLab/db-sample-schemas-19c/sales_history/ /home/oracle/ v3 localhost:1521/pdb1
    </copy>
    ````

    ![schema installed](./images/schemas-created.png " " )
   
7.  Exit SQL Plus to the oracle user.

    ```
    <copy>
    exit
    </copy>
    ```

    ![return to opc](images/return-to-opc.png)

7.  Create gzip backup of the sample data datafile (storage compression simulation)
    ````
    <copy>
    du -hs /u01/oradata/cdb1/pdb1/test_data.dbf
    cp /u01/oradata/cdb1/pdb1/test_data.dbf /u01/oradata/test_data.dbf
    </copy>
    ````
    ![return to opc](images/backup-test-data-file.png)

    ````
    <copy>
    gzip /u01/oradata/test_data.dbf
    du -hs /u01/oradata/test_data.dbf.gz
    </copy>
    ````
    ![return to opc](images/storage-compression-simulation.png)

**Note: As you can see, we just backed up the test_data.dbf datafile and compressed it from 311MB to 14M.** 

Congratulations! Now you have the environment to run the labs.

You may now **proceed to the next lab**.

## Oracle Database Sample Schemas Agreement

Copyright (c) 2019 Oracle

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

*THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.*

## **Acknowledgements**

- **Author** - Royce Fu
- **Contributors** - Noah Galloso, Richard Evans
* **Last Updated By/Date** - Valentin Tabacaru, Mar 2023