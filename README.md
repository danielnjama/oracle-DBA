# Oracle DBA - Introduction

Welcome to the Oracle DBA (Database Administrator) course. This module introduces foundational database concepts, the responsibilities of a DBA, and key Oracle architecture terms. Understanding these basics is essential before diving into Oracle Database internals and administration tasks.

---

## Table of Contents

1. [What is a Database?](#what-is-a-database)
2. [Role of a DBA (Database Administrator)](#role-of-a-dba-database-administrator)
3. [OLTP vs OLAP Databases](#oltp-vs-olap-databases)
4. [Database ABCs (Terminologies)](#database-abcs-terminologies)
5. [Undo and Redo Data](#undo-and-redo-data)
6. [Oracle Instance vs Database](#oracle-instance-vs-database)
7. [Oracle Database Architecture (High-level Overview)](#oracle-database-architecture-high-level-overview)
8. [Conclusion](#conclusion)

---

## What is a Database?

A **database** is an organized collection of data, generally stored and accessed electronically from a computer system. In the context of Oracle, it refers to a structured environment where data is stored in tables, and can be retrieved, manipulated, and managed efficiently using SQL.

### Characteristics:
- Structured data storage
- Data integrity and security
- Concurrent access
- Scalability and recovery options

---

## Role of a DBA (Database Administrator)

A **DBA** is responsible for the installation, configuration, upgrade, administration, monitoring, and maintenance of databases in an organization.

### Key Responsibilities:
- Install and configure Oracle software
- Manage database storage structures
- Monitor performance and tune databases
- Backup and recovery
- Implement security measures
- Handle user access and privileges
- Plan for disaster recovery

---

## OLTP vs OLAP Databases

| Feature            | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|--------------------|--------------------------------------|-------------------------------------|
| Purpose            | Day-to-day transactions              | Data analysis and reporting         |
| Data Operations    | Insert, Update, Delete               | Read (Select)                       |
| Data Volume        | Large number of short transactions   | Fewer, complex queries              |
| Examples           | Banking systems, e-commerce          | BI tools, data warehousing          |
| Normalization      | Highly normalized                    | De-normalized for performance       |

---

## Database ABCs (Terminologies)

- **Schema**: Logical structure that groups database objects like tables, views, and procedures.
- **Table**: Collection of rows and columns storing data.
- **Row (Record)**: A single entry in a table.
- **Column (Field)**: Defines the data type of a value in a row.
- **SQL**: Structured Query Language used for accessing and manipulating data.
- **PL/SQL**: Oracle’s procedural extension for SQL.

---

## Undo and Redo Data

### Undo Data:
- Stores **before images** of data to support:
  - Rollback operations
  - Read-consistent queries
  - Flashback features

### Redo Data:
- Records **after images** of changes made to the database.
- Stored in **redo log files**
- Used for:
  - Crash recovery
  - Data synchronization in Data Guard

---

## Oracle Instance vs Database

### Oracle Instance:
- A combination of **memory structures (SGA)** and **background processes**.
- It manages database operations.

### Oracle Database:
- A set of physical files (datafiles, control files, redo log files) that store data.

> 📌 **Note:** An instance must be started before accessing the database.

| Component     | Oracle Instance                   | Oracle Database                   |
|---------------|------------------------------------|------------------------------------|
| Purpose       | Manages access to DB               | Stores data persistently           |
| Components    | SGA, Background processes          | Datafiles, Redo logs, Control files|
| Lifespan      | Exists in memory (volatile)        | Stored on disk (non-volatile)      |

---

## Oracle Database Architecture (High-level Overview)

- **SGA (System Global Area)**: Shared memory area containing data and control information.
- **Background Processes**: Includes DBWn (Database Writer), LGWR (Log Writer), CKPT (Checkpoint), SMON (System Monitor), PMON (Process Monitor), ARCn (Archiver), etc.
- **Datafiles**: Store actual user and system data.
- **Control File**: Maintains metadata about the database structure.
- **Redo Log Files**: Record changes for recovery purposes.
- **Undo Segments**: Store undo data for rollback and consistency.

---


# Oracle DBA - Topic 2: Oracle Process Architecture

Oracle Database operates through a combination of memory structures and background processes. The process architecture is a core component of the Oracle Instance, playing a vital role in database operations like writing data to disk, recovery, log management, and monitoring.

---

## Overview of Oracle Process Architecture

When an Oracle database is started, an **Oracle instance** is created, consisting of:
- **SGA (System Global Area)** – Shared memory
- **Background Processes** – Manage internal tasks

Oracle uses **server processes** to handle client requests and **background processes** to manage core operations.

---

## Types of Processes

Oracle processes can be grouped as:

- **Client/Server Processes**:
  - User processes
  - Server processes (dedicated/shared)
  
- **Background Processes**:
  - Mandatory processes (e.g., DBWn, LGWR, SMON, PMON)
  - Optional processes (e.g., ARCn, RECO, CJQ0)

---

## Database Writer (DBWn)

- Writes modified (dirty) data blocks from **DB buffer cache** to datafiles.
- Helps free up space in the buffer cache.
- Multiple DBWn processes may exist in large databases (DBW0, DBW1, ...).

📌 **Trigger Conditions**:
- Buffer cache becomes full
- Checkpoint occurs
- Tablespace is placed in read-only mode
- Database is shut down cleanly

---

## Log Writer (LGWR)

- Writes **redo entries** from **redo log buffer** in SGA to **online redo log files**.
- Ensures **recoverability** of committed transactions.
- Writes synchronously during:
  - COMMIT
  - Log buffer 1/3 full
  - Every 3 seconds
  - Before DBWn writes dirty blocks

---

## Checkpoint (CKPT)

- Signals DBWn to write all dirty buffers to disk.
- Updates control files and datafile headers with checkpoint info.

📌 **Purpose**:
- Reduces instance recovery time
- Ensures consistency between memory and disk

---

## System Monitor (SMON)

- Performs **instance recovery** on startup after an abnormal shutdown.
- Coalesces free space in tablespaces.
- Cleans up temporary segments no longer in use.

---

## Process Monitor (PMON)

- Cleans up resources from **failed user processes** (e.g., locks, memory).
- Registers the instance with the **listener**.

📌 **Note**: Automatically restarts failed server or dispatcher processes.

---

## Recoverer Process (RECO)

- Handles the resolution of **distributed transactions**.
- Automatically recovers **in-doubt transactions** in distributed databases.

---

## Archiver Process (ARCn)

- Copies redo log files to **archive destinations** after a log switch (if archive log mode is enabled).
- Supports **media recovery**, **Data Guard**, and **backup** strategies.

📌 You can configure multiple ARCn processes for high-volume databases.

---

## Other Background Processes

| Process  | Description |
|----------|-------------|
| **CJQ0** | Job queue coordinator, manages scheduled jobs. |
| **MMON** | Monitors DB health and performs AWR statistics gathering. |
| **MMNL** | Lightweight MMON for less-critical statistics tasks. |
| **DBRM** | Manages Database Resource Manager. |
| **MMAN** | Manages memory components for automatic memory management. |
| **LREG** | Registers and updates services with the listener. |
| **AQPC** | Manages Advanced Queuing processes. |
| **RVWR** | Writes flashback logs (used with Flashback Database). |

---



# Oracle DBA - Topic 3: Oracle Memory Architecture

Oracle Database's performance heavily depends on efficient memory management. The Oracle memory architecture is divided into two major components: **System Global Area (SGA)** and **Program Global Area (PGA)**. This module explains each component and its substructures in detail.

---

## Table of Contents

## Overview of Memory Architecture

Oracle's memory structure includes:

- **SGA** – Shared memory for all users.
- **PGA** – Private memory for individual server processes.

Efficient tuning and understanding of these areas are key to optimal database performance.

---

## System Global Area (SGA)

The **SGA** is a shared memory region that contains data and control information for the Oracle instance. It is allocated at instance startup.

### Key Components:

---

### Database Buffer Cache

- Stores copies of data blocks read from datafiles.
- Improves performance by reducing disk I/O.
- Data modifications happen in the buffer cache before being written to disk by **DBWn**.

### Buffer Cache Types:
- **Default Cache**: Standard data blocks.
- **Keep Buffer Pool**: Keeps frequently accessed blocks longer.
- **Recycle Buffer Pool**: Holds blocks briefly.
- **nK Buffer Cache**: Used for tablespaces with non-default block sizes (2K, 4K, 8K, 16K, 32K).

---

### Shared Pool

- Caches SQL and PL/SQL execution plans and metadata.

#### Subcomponents:
- **Library Cache**: Stores parsed SQL and PL/SQL.
- **Data Dictionary Cache**: Stores metadata like table definitions and user privileges.

📌 **Benefit**: Avoids repeated parsing of SQL statements.

---

### Redo Log Buffer

- Temporarily stores redo entries generated by DML and DDL operations.
- Contents are periodically written to **redo log files** by **LGWR**.

---

### Large Pool

- Optional memory area for:
  - Shared server connections
  - Backup and restore operations
  - Parallel execution

> 📌 Helps reduce contention in the shared pool.

---

### Java Pool & Streams Pool

- **Java Pool**: Memory for Java code execution within the database (e.g., Java stored procedures).
- **Streams Pool**: Used for Oracle Streams and Advanced Queuing for replication or message queuing.

---

### Keep Buffer Pool

- Keeps selected data blocks in memory longer than default.
- Ideal for frequently accessed tables needing high cache retention.

---

### Recycle Buffer Pool

- Used for data blocks that are rarely accessed.
- Reduces pressure on default buffer cache.

---

### Non-Default (nK) Buffer Cache

- Separate buffer cache areas for tablespaces using **non-default block sizes**.
- Allows multiple block sizes in a single instance for optimized I/O.

---

## Program Global Area (PGA)

The **PGA** is memory used by **dedicated server processes** for:
- Session memory
- Sort area
- Hash joins
- Bitmap merges
- Private SQL areas

📌 **PGA is not shared**; it is allocated per user session.

---

## Dedicated vs Shared Server Architecture

| Feature         | Dedicated Server                | Shared Server                      |
|------------------|----------------------------------|-------------------------------------|
| Connection       | One server process per session  | Multiple sessions share server processes |
| Memory Usage     | More PGA per session            | More SGA usage, less PGA per session |
| Scalability      | Limited by OS process limits    | Scales better with many users       |

---

## PGA in Shared Server Environment

- In shared server mode, **PGA usage is reduced** since user sessions share server processes.
- Some parts of the session memory are moved to **SGA (UGA - User Global Area)**.
- UGA is stored in **large pool** (if configured) or **shared pool**.

📌 **Important Note**: In shared server mode, tuning **SGA** becomes more critical than PGA.

---

# Oracle DBA - Topic 4: Software Installation & Database Creation

In this module, you’ll learn how to install Oracle software, create a database manually and using DBCA, verify the database setup, understand different shutdown modes, and perform an Oracle 12c upgrade.

---

## Oracle Software Installation

Oracle software can be installed via:

### GUI-Based Installer:
1. Download the Oracle Database software from Oracle's official site.
2. Unzip and run `runInstaller`.
3. Choose installation type: **Server Class**, **Desktop Class**, etc.
4. Set Oracle base, software location, and other parameters.
5. Proceed through configuration screens.
6. Execute scripts as prompted (`root.sh`).

### Silent Mode (Command Line):
- Use a response file for automated installations.

```bash
./runInstaller -silent -responseFile /path/to/db_install.rsp
```

## Database Creation - Manual Method
### 1. Set Oracle environment variables
```bash
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
```

### 2. Create Initialization Parameter File
Example: initORCL.ora
```bash
db_name=ORCL
memory_target=1G
control_files=(/u01/oradata/ORCL/control01.ctl)
```

### 3. Start Instance in NOMOUNT Mode
```bash
sqlplus / as sysdba
STARTUP NOMOUNT PFILE='/path/to/initORCL.ora';
```

### 4. Create Database
```bash
CREATE DATABASE ORCL
   USER SYS IDENTIFIED BY oracle
   USER SYSTEM IDENTIFIED BY oracle
   LOGFILE GROUP 1 ('/u01/oradata/ORCL/redo01.log') SIZE 50M,
           GROUP 2 ('/u01/oradata/ORCL/redo02.log') SIZE 50M
   MAXLOGFILES 5
   MAXDATAFILES 100
   CHARACTER SET AL32UTF8
   DATAFILE '/u01/oradata/ORCL/system01.dbf' SIZE 700M REUSE
   SYSAUX DATAFILE '/u01/oradata/ORCL/sysaux01.dbf' SIZE 550M
   DEFAULT TEMPORARY TABLESPACE temp
      TEMPFILE '/u01/oradata/ORCL/temp01.dbf' SIZE 20M
   UNDO TABLESPACE undotbs1
      DATAFILE '/u01/oradata/ORCL/undotbs01.dbf' SIZE 200M;
```

### 5. Create Data Dictionary Views
```bash
@?/rdbms/admin/catalog.sql
@?/rdbms/admin/catproc.sql
```

### Verifying Database Creation
- Check the open status:
```sql
SELECT name, open_mode FROM v$database;
```
- Validate files:
```bash
ls -l /u01/oradata/ORCL/
```
- Check listener registration:
```bash
lsnrctl status
```
- Other checks to validate if database is open.
```
### Method 1
#Access the database
sqlplus / as sysdba

#then
SELECT status FROM v$instance;

If the instance is not started, this will fail.

If started but the DB is not mounted or open, you'll see:

- STARTED
- MOUNTED
- OPEN

### Method 2
-- Check instance status
SELECT status FROM v$instance;

-- If needed, start instance
STARTUP NOMOUNT;

-- Mount the database (optional if not open yet)
ALTER DATABASE MOUNT;

-- Open the database
ALTER DATABASE OPEN;

### Method 3
# On Linux/Unix
ps -ef | grep pmon

## If you get something like below; the database is up and running.
oracle   1234     1  0 10:00 ?  00:00:00 ora_pmon_ORCL
```

## Database Creation using DBCA
DBCA (Database Configuration Assistant) is a GUI tool to create/configure databases.
### Steps:
1. Set display support. Run  xhost +
2. Run dbca
3. Choose Create Database
4. Select database type (Single Instance / RAC / Container DB)
5. Configure:
    - Global DB name & SID
    - Storage locations
    - Memory & sizing
    - Character set
    - Sample schemas (optional)
6. Review and complete the process

## Silent mode
```bash
dbca -silent -createDatabase \
     -templateName XE_Database.dbc \
     -gdbname ORCL -sid ORCL \
     -responseFile NO_VALUE \
     -characterSet AL32UTF8 \
     -memoryPercentage 40 \
     -emConfiguration NONE \
     -sysPassword oracle \
     -systemPassword oracle
```
Note:
To locate the templateName: do this:
```bash
ls $ORACLE_HOME/assistants/dbca/templates/
```
Check for a file that ends with dbc.  Common template names are:
- General_Purpose.dbc
- Data_Warehouse.dbc
- New_Database.dbt (for custom templates)

# Oracle Shutdown Methods

Oracle Database supports multiple shutdown methods, each suited to different scenarios depending on the urgency and impact on users and transactions.

## 🔻 Shutdown Methods

| Method        | Description |
|---------------|-------------|
| **NORMAL**      | Waits for all users to disconnect. No new connections are allowed during this time. Ideal for planned maintenance. |
| **IMMEDIATE**   | Terminates active transactions, rolls back any uncommitted changes, and shuts down the database cleanly. Recommended for most shutdowns. |
| **TRANSACTIONAL** | Waits for active transactions to complete before disconnecting users and shutting down. Useful when you want to preserve user work in progress. |
| **ABORT**        | Forces an immediate shutdown without rolling back transactions or performing cleanup. Should only be used when other shutdown methods fail. |

## Usage in SQL*Plus

Connect as SYSDBA and run one of the following:

```sql
SHUTDOWN NORMAL;
SHUTDOWN IMMEDIATE;
SHUTDOWN TRANSACTIONAL;
SHUTDOWN ABORT;
```



# Oracle 12c Upgrade Overview
----- notes ---



# Oracle DBA - Topic 5: Tablespace Management

Tablespaces are logical storage containers in Oracle that allow efficient data organization and management. This module explains how Oracle stores data, how to manage tablespaces, types of tablespaces, and various advanced features like temporary tablespace groups and extent management.

---

## Table of Contents
## How Data is Stored in a Database

- Data in Oracle is stored **physically in datafiles** and **logically in tablespaces**.
- Tablespaces contain **segments**, which are made of **extents**, and extents are made of **blocks**.
- This hierarchy allows Oracle to manage data efficiently at different levels.

---

## Tablespace Concepts

- **Tablespace**: A logical container that holds schema objects like tables and indexes.
- Each tablespace is associated with one or more **datafiles**.
- A single datafile belongs to one and only one tablespace.

---

## Types of Tablespaces

### Permanent Tablespaces
- Store permanent user and system data.
- Example: `USERS`, `SYSTEM`, `SYSAUX`

### Temporary Tablespaces
- Store intermediate results of operations like sorts, joins, and groupings.
- Data is not persistent and is cleared after the session ends.

### Undo Tablespaces
- Store undo data used to roll back transactions and recover the database.

### Bigfile vs Smallfile Tablespaces

| Type       | Description                            |
|------------|----------------------------------------|
| Smallfile  | Default. Allows multiple datafiles.     |
| Bigfile    | Contains one very large datafile. Useful in large databases, easier management via ASM or OMF.|

---

## Online vs Offline Tablespaces

| State      | Description |
|------------|-------------|
| **Online** | Available for read/write operations. |
| **Offline**| Unavailable to users; useful for maintenance or data recovery. |

```sql
-- Make offline
ALTER TABLESPACE users OFFLINE;

-- Bring back online
ALTER TABLESPACE users ONLINE;
```

## Creating a Tablespace
```sql
CREATE TABLESPACE mydata
DATAFILE '/u01/oradata/mydata01.dbf' SIZE 100M
AUTOEXTEND ON NEXT 10M MAXSIZE 1G
EXTENT MANAGEMENT LOCAL
SEGMENT SPACE MANAGEMENT AUTO;
```
EXTENT MANAGEMENT LOCAL is preferred over dictionary-managed (obsolete from 10g onward).

### Tablespace with Different Block Sizes
- Useful when certain workloads perform better with non-default block sizes.
- Default block size is set at DB creation.
- Other block sizes: 2K, 4K, 8K, 16K, 32K.

### Example (8K DB default, create 16K tablespace):
```sql
CREATE TABLESPACE data_16k
DATAFILE '/u01/oradata/data_16k.dbf' SIZE 100M
BLOCKSIZE 16K
EXTENT MANAGEMENT LOCAL;
```

Requires a matching DB_nK_CACHE_SIZE initialization parameter.

### Temporary Tablespace Management
```sql
CREATE TEMPORARY TABLESPACE temp_data
TEMPFILE '/u01/oradata/temp_data01.dbf' SIZE 100M
EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M;
```
- Temporary tablespaces use tempfiles, not datafiles.
- Can’t contain permanent segments.

### Temporary Tablespace Groups
- Combine multiple temporary tablespaces into one logical unit.
- Improves scalability for sorting operations across multiple tempfiles.

### Create and Add:
```sql
CREATE TEMPORARY TABLESPACE temp1 TEMPFILE '/u01/temp1.dbf' SIZE 50M;
CREATE TEMPORARY TABLESPACE temp2 TEMPFILE '/u01/temp2.dbf' SIZE 50M;

ALTER DATABASE DEFAULT TEMPORARY TABLESPACE temp1;

ALTER TABLESPACE temp2 TABLESPACE GROUP temp_group1;
ALTER TABLESPACE temp1 TABLESPACE GROUP temp_group1;
```

### Assign to User:
```sql
ALTER USER scott TEMPORARY TABLESPACE temp_group1;
```
## Extent Management
  - Local Extent Management:

    - Tracks extents using bitmaps.
    - More efficient than old dictionary-based tracking.
    - Automatically manages extent allocation.
  - Extent Allocation Types:
    - Uniform: All extents are same size.
    - Autoallocate: Oracle decides extent sizes.

Example:
```sql
-- Uniform extent size
CREATE TABLESPACE uniform_ts
DATAFILE '/u01/uniform01.dbf' SIZE 100M
EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M;

-- Autoallocate
CREATE TABLESPACE auto_ts
DATAFILE '/u01/auto01.dbf' SIZE 100M
EXTENT MANAGEMENT LOCAL AUTOALLOCATE;
```

# Oracle Key File Locations and Retrieval Queries

During Oracle database backups, it is essential to know where critical files are located on disk. This README provides SQL queries to retrieve the paths of these key components directly from the database.

---

##  Key Oracle File Types and Their Queries

| Component           | Description                                                  | Query |
|---------------------|--------------------------------------------------------------|-------|
| **Datafiles**        | Store user and system data (tables, indexes, etc.)           | `SELECT file_name FROM dba_data_files;` |
| **Tempfiles**        | Used for sorting and temporary operations                    | `SELECT file_name FROM dba_temp_files;` |
| **Redo Log Files**   | Track all database changes for recovery                      | `SELECT member FROM v$logfile;` |
| **Control Files**    | Store database metadata and structure                        | `SELECT name FROM v$controlfile;` |
| **SPFILE / PFILE**   | Initialization parameter files used to start the database    | `SHOW PARAMETER spfile;`<br>`SHOW PARAMETER pfile;` |
| **Archive Log Files**| Store redo logs when the DB is in ARCHIVELOG mode            | `SHOW PARAMETER log_archive_dest;` |
| **Flash Recovery Area (FRA)** | Optional location for backups and archived logs     | `SHOW PARAMETER db_recovery_file_dest;` |

---

##  Sample Usage in SQL\*Plus

```sql
-- Connect as SYSDBA
sqlplus / as sysdba

-- Query datafile locations
SELECT file_name FROM dba_data_files;

-- Query control file locations
SELECT name FROM v$controlfile;

-- Query redo log file locations
SELECT member FROM v$logfile;

-- Show archive log destination
SHOW PARAMETER log_archive_dest;

-- Show parameter file (spfile or pfile)
SHOW PARAMETER spfile;
```


# Oracle DBA - Topic 6: Undo Management

Undo is a critical part of Oracle’s transaction processing. It stores information required to rollback transactions, recover the database, and provide read consistency.

---
## What is Undo?

Undo is the data Oracle stores to:
- Roll back uncommitted changes when a transaction is aborted.
- Provide **read consistency** by supplying previous versions of data.
- Recover the database to a consistent state after a failure.

---

## Purpose of Undo Data

- **Transaction Rollback** – Undo allows partial or full rollback of a transaction.
- **Read Consistency** – Ensures users don’t see intermediate or uncommitted changes.
- **Database Recovery** – During recovery, undo helps Oracle roll back uncommitted changes.
- **Flashback Features** – Flashback queries and flashback table operations use undo data.

---

## Undo Segments and Tablespaces

- **Undo Segments**: Special segments that store undo records for transactions.
- **Undo Tablespace**: Dedicated tablespace where undo segments reside (from Oracle 9i onwards).

---

## Undo Management Types

### Automatic Undo Management (AUM)
- Oracle automatically creates and manages undo segments.
- Recommended and default mode since Oracle 9i.
- Controlled by `UNDO_MANAGEMENT=AUTO`.

### Manual Undo Management
- DBA manually creates and manages rollback segments.
- Deprecated in newer Oracle versions.
- Controlled by `UNDO_MANAGEMENT=MANUAL`.

---

## Undo Retention

- **UNDO_RETENTION** parameter defines how long (in seconds) undo data should be retained.
- Helps with long-running queries and flashback operations.

```sql
ALTER SYSTEM SET UNDO_RETENTION = 1800; -- 30 minutes
```

To see current retention period
```sql
SHOW PARAMETER undo_retention;


OR
SELECT name, value
FROM v$parameter
WHERE name = 'undo_retention';
```

## Guaranteed Undo Retention
- Ensures undo data is retained for the specified retention period, even if space is needed.
```sql
ALTER TABLESPACE undotbs1 RETENTION GUARANTEE;
```

- Default behavior is NOGUARANTEE, where undo may be overwritten if needed.


## Managing Undo Tablespace
Creating a New Undo Tablespace:
```sql
CREATE UNDO TABLESPACE undotbs2
DATAFILE '/u01/oradata/undotbs2.dbf' SIZE 500M
AUTOEXTEND ON;
```
### Setting Undo Tablespace for Use:
```sql
ALTER SYSTEM SET UNDO_TABLESPACE = undotbs2;
```
### Dropping an Undo Tablespace:
- Only possible when not in use:
```sql
DROP TABLESPACE undotbs1 INCLUDING CONTENTS AND DATAFILES;
```
### Switching Undo Tablespace
1. Create new undo tablespace (if not exists).
2. Set the new tablespace as active:
```sql
ALTER SYSTEM SET UNDO_TABLESPACE = undotbs2;
```
3. Optional: Drop the old undo tablespace.

### Monitoring Undo Usage
#### Check Active Undo Tablespace:
```sql
SHOW PARAMETER undo_tablespace;
```

### View Undo Statistics:
```sql
SELECT a.tablespace_name, b.undo_size, a.status
FROM dba_tablespaces a, v$undostat b
WHERE a.contents = 'UNDO';
```

### Check Undo Usage Per Transaction:
```sql
SELECT s.sid, s.serial#, t.used_urec, t.used_ublk
FROM v$transaction t, v$session s
WHERE t.ses_addr = s.saddr;
```

### Best Practices
- Use Automatic Undo Management.
- Set a reasonable UNDO_RETENTION based on workload.
- Enable RETENTION GUARANTEE if using flashback features.
- Monitor v$undostat to tune undo size and retention.
- Avoid using manual undo management unless absolutely required.


# Oracle DBA - Redo Management

Redo management is a core component of Oracle's architecture that ensures transactional durability, recovery capability, and data consistency. Every change to the database is first written to **redo logs**, making redo crucial for crash recovery and replication.

---

## What is Redo?

- Redo records all changes made to database objects, including data and metadata.
- It includes INSERTs, UPDATEs, DELETEs, and structural changes like table creation.
- Redo logs help Oracle recover from:
  - Instance failure
  - Media failure
  - System crash

---

## Core Components of Redo

| Component       | Description |
|----------------|-------------|
| **Redo Log Buffer** | Memory area in SGA storing redo entries before writing to disk. |
| **Online Redo Log Files** | Disk files where redo data is persistently stored. |
| **Log Writer (LGWR)** | Background process that writes from redo buffer to redo files. |
| **Redo Log Groups** | Logical collections of redo log members. |
| **Redo Log Members** | Physical redo files in a group (multiplexed for safety). |

---

## How Redo Works (Flow)

1. A user changes data (e.g., `UPDATE` statement).
2. Redo record generated and placed into the **redo buffer**.
3. **LGWR** writes redo entries from buffer to online redo logs:
   - On COMMIT
   - Every 3 seconds
   - When redo buffer is one-third full
   - Before **DBWR** flushes dirty blocks
4. Changes are now durable and recoverable.

---

## Redo Log Buffer

- Part of the **System Global Area (SGA)**.
- Size controlled by `LOG_BUFFER` parameter.
- Holds redo records until flushed by LGWR.

```sql
SHOW PARAMETER log_buffer;
```

### Redo Log Groups & Members
- Oracle requires at least 2 log groups for circular writing.
- Each group can have one or more members (physical files).
- Multiple members = multiplexing, improves fault tolerance.

### Creating Redo Log Groups
```sql
ALTER DATABASE ADD LOGFILE GROUP 3 (
  '/u01/app/oracle/oradata/ORCL/redo03a.log',
  '/u02/app/oracle/oradata/ORCL/redo03b.log'
) SIZE 100M;
```

### Adding Redo Log Member
```sql
ALTER DATABASE ADD LOGFILE MEMBER '/u03/oradata/redo01c.log' TO GROUP 1;
```

### Log Switches
- A log switch occurs when Oracle finishes writing to one redo group and moves to the next.
- It happens automatically or can be forced manually.

### Manual Switch
```sql
ALTER SYSTEM SWITCH LOGFILE;
```
### Monitoring Redo Logs
#### View Redo Log Groups
```sql
SELECT GROUP#, STATUS, BYTES/1024/1024 AS SIZE_MB FROM V$LOG;
```
#### View Redo Log Members
```sql
SELECT GROUP#, MEMBER, TYPE FROM V$LOGFILE;
```

# Oracle DBA - User Management

User management in Oracle involves creating, modifying, and controlling access for database users. DBAs are responsible for user creation, granting roles and privileges, controlling resource usage, and ensuring database security.

---

##  What is a User in Oracle?

- A **user** is a database account that can own schema objects and connect to the database.
- Each user has a **default tablespace**, **temporary tablespace**, **profile**, and **authentication mechanism**.

---

##  Creating a User

### Syntax:

```sql
CREATE USER username IDENTIFIED BY user_password
DEFAULT TABLESPACE users
TEMPORARY TABLESPACE temp
QUOTA 100M ON users;
```

```sql
CREATE USER username IDENTIFIED BY user_password
password expire;
```
OR simply
```sql
CREATE USER username IDENTIFIED BY user_password
```
- IDENTIFIED BY: Sets the password.

- DEFAUL9T TABLESPACE: Location where user’s objects are stored.

- TEMPORARY TABLESPACE: Used for sort operations.

- QUOTA: Limits how much space a user can use in a tablespace.


### Granting Privileges
Oracle has two types of privileges:

1. System Privileges – e.g., CREATE TABLE, CREATE USER, ALTER SESSION
2. Object Privileges – e.g., SELECT, INSERT, UPDATE on specific objects

### Granting System Privileges:
```sql
GRANT CREATE SESSION, CREATE TABLE TO john;
```

### Granting Object Privileges:
```sql
GRANT SELECT, INSERT ON employees TO john;
```

## Roles
- A role is a collection of privileges.
- Makes privilege management easier.

### Creating and Granting Roles
```sql
CREATE ROLE hr_role;
GRANT SELECT, INSERT ON employees TO hr_role;  #employees table
GRANT hr_role TO john;
```

### Default and Admin Options:
```sql
GRANT hr_role TO john WITH ADMIN OPTION;
```

## Viewing Users and Privileges

### List All Users
```sql
SELECT username, account_status FROM dba_users;
```

### Show System Privileges
```sql
SELECT * FROM dba_sys_privs WHERE grantee = 'JOHN';
```

### Show Object Privileges
```sql
SELECT * FROM dba_tab_privs WHERE grantee = 'JOHN';
```

## User Authentication and Locking
### Authentication types:
- Password
- External (OS)
- Global (LDAP)

### Locking and Unlocking Users
```sql
ALTER USER john ACCOUNT LOCK;
ALTER USER john ACCOUNT UNLOCK;
```


## Password Expiry and Profiles
- Use profiles to enforce password policies and resource limits.

### Creating a Profile
```sql
CREATE PROFILE limited_user LIMIT
  FAILED_LOGIN_ATTEMPTS 5
  PASSWORD_LIFE_TIME 30;
```
### Assign Profile to User
```sql
ALTER USER john PROFILE limited_user;
```

## Altering and Dropping Users
### Change User Password
```sql
ALTER USER john IDENTIFIED BY new_password;
```
### Change Default Tablespace
```sql
ALTER USER john DEFAULT TABLESPACE example;
```

### Drop a User
```sql
DROP USER john;
OR
DROP USER john CASCADE;
```
- CASCADE is required to drop user-owned schema objects.

### Common User vs Local User (Multitenant - CDB/PDB)
- Common User: Exists in all containers (prefixed with C##)
- Local User: Exists only in one pluggable database (PDB)

### Create Common User
```sql
CREATE USER C##admin IDENTIFIED BY password CONTAINER=ALL;
```
### Create Local User
```sql
CREATE USER app_user IDENTIFIED BY secret CONTAINER=CURRENT;
```


# Oracle Multitenant Architecture

## Overview

Oracle Database 12c and later introduced **Multitenant Architecture**, which uses:

- **CDB (Container Database)**: The root container that holds system metadata and can contain multiple **PDBs**.
- **PDB (Pluggable Database)**: A self-contained database that runs inside a CDB. It has its own users, schemas, and application data.

---

## CDB vs PDB: Key Differences

| Feature               | CDB (CDB$ROOT)                   | PDB (e.g., ORCLPDB1)                   |
|-----------------------|----------------------------------|----------------------------------------|
| Purpose               | System-level operations          | Application-specific data              |
| Users/Roles           | Common (start with `C##`)        | Local (no special prefix)              |
| Tables/Schemas        | Not recommended                  | ✅ Create tables/schemas here          |
| Isolation             | Shared across PDBs               | Isolated to each PDB                   |
| Data Access           | No app data                      | Full control over user/app data        |

---

## 🛠️ DBA Tasks: Where to Do What

| Task                           | Recommended Container |
|--------------------------------|------------------------|
| Create application tables      | ✅ PDB                 |
| Create users and roles         | ✅ PDB (Local Users)   |
| Common users (across all PDBs) | ❌ CDB (use `C##`)     |
| Grant privileges               | ✅ PDB                 |
| Install applications           | ✅ PDB                 |
| System-level config (e.g. undo)| ✅ CDB                 |

---

## 🔍 Checking and Managing Containers

### ✅ Show Current Container
```sql
SHOW CON_NAME;
```
### List All Containers
```sql
SELECT CON_ID, NAME, OPEN_MODE, FROM V$CONTAINERS;
```
### Switch to Another PDB
```sql
ALTER SESSION SET CONTAINER = your_pdb_name;
```
Or connect directly:
```bash
sqlplus username/password@your_pdb_name
```

### Creating Users and Roles
```sql
## Create a Local User in a PDB
-- Must be connected to the PDB
CREATE USER hr IDENTIFIED BY hrpass;
GRANT CONNECT, RESOURCE TO hr;

## Create a Common User (Only in CDB)
-- Must be connected to CDB$ROOT
CREATE USER C##admin IDENTIFIED BY password;
GRANT DBA TO C##admin CONTAINER=ALL;
```
## PDB Management
### List All PDBs
```sql
SELECT NAME, OPEN_MODE FROM V$PDBS;
```
### Open a PDB
```sql
ALTER PLUGGABLE DATABASE your_pdb_name OPEN;
```
### Close a PDB
```sql
ALTER PLUGGABLE DATABASE your_pdb_name CLOSE IMMEDIATE;
```
### Useful Views
| View           | Description               |
| -------------- | ------------------------- |
| `V$CONTAINERS` | Lists all containers      |
| `V$PDBS`       | Lists PDB info            |
| `CDB_USERS`    | All users in CDB and PDBs |
| `CDB_TABLES`   | Tables across containers  |
| `DBA_ROLES`    | All defined roles         |

### Best Practices
- Always operate in a PDB for application/database work.
- Use CDB only for container-level configurations and shared resources.
- Avoid creating application tables or users directly in the CDB$ROOT.
- Name common users/roles with C## prefix only when needed.
- Regularly check which container you're in before executing critical commands.


# Oracle DBA - Backup and Recovery

Backup and recovery are core responsibilities of a DBA. They ensure data availability, integrity, and durability in case of hardware failure, user error, or data corruption.

---

##  What is Backup?

A **backup** is a copy of data from the database stored separately to allow recovery in case of data loss.

### Types of Backups:

| Type        | Description |
|-------------|-------------|
| **Cold Backup** (Offline) | Taken when DB is shut down. Files are copied at OS level. |
| **Hot Backup** (Online)  | Taken while DB is open. Requires ARCHIVELOG mode. |
| **RMAN Backup**          | Oracle’s recommended method using Recovery Manager. |

---

##  What is Recovery?

**Recovery** is the process of restoring lost or damaged data using backup files and applying redo data to bring the DB to a consistent state.

### Recovery Types:

| Type                  | Description |
|-----------------------|-------------|
| **Instance Recovery** | Handled automatically by SMON after a crash. |
| **Media Recovery**    | Required when physical files are lost/corrupted. |
| **Incomplete Recovery** | Recovers up to a point in time (PITR), SCN, or log sequence. |

---

##  Physical vs Logical Backup

| Physical Backup        | Logical Backup          |
|------------------------|-------------------------|
| Uses datafiles, control files, redo logs | Uses `exp` / `expdp` for schema/rows |
| RMAN or OS copy        | Data Pump utilities     |
| Faster recovery        | Good for migrations     |

---

##  Backup Components

- **Datafiles**: Store actual data
- **Control File**: Tracks structure of DB
- **Redo Logs**: Track changes (for recovery)
- **Archived Redo Logs**: Needed for media recovery
- **Parameter Files**: SPFILE or PFILE

---

##  Cold Backup Steps

1. Shutdown database:

```sql
SHUTDOWN IMMEDIATE;
```
2. Copy datafiles, control files, and redo logs at OS level.
3. Startup database:
```sql
STARTUP;
```
- Cold backup is consistent but requires downtime.


## Hot Backup (User-managed)
1. Enable ARCHIVELOG mode:
```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
```
2. Begin tablespace backup:
```sql
ALTER TABLESPACE users BEGIN BACKUP;
-- Copy datafile
ALTER TABLESPACE users END BACKUP;
```
3. Backup control and redo logs:
```sql
ALTER DATABASE BACKUP CONTROLFILE TO '/backup/control01.ctl';
```

## RMAN (Recovery Manager)
RMAN automates and manages backup/recovery operations.

## Configure RMAN:
```sql
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

## Full DB Backup:
```sql
RMAN> BACKUP DATABASE;
```

## Backup with Archive Logs:
```sql
RMAN> BACKUP DATABASE PLUS ARCHIVELOG;
```

### Backup Report Queries
check backup info:
```sql
SELECT * FROM V$BACKUP;
SELECT * FROM V$DATAFILE;
SELECT * FROM V$ARCHIVED_LOG;
```


##  Restore and Recovery Using RMAN

### Restore from RMAN Backup

```bash
RMAN> SHUTDOWN IMMEDIATE;
RMAN> STARTUP MOUNT;
RMAN> RESTORE DATABASE;
RMAN> RECOVER DATABASE;
RMAN> ALTER DATABASE OPEN;
```
If a control file is lost, restore it first using RESTORE CONTROLFILE.

## Control File and SPFILE Backup
### Backup Control File
```bash
RMAN> BACKUP CURRENT CONTROLFILE;
```

### Backup SPFILE
```sql
RMAN> BACKUP SPFILE;
```
Control file backups are important for restoring DB structure and log history.


## Incomplete Recovery (Point-in-Time Recovery - PITR)
Used when recovery is needed up to a specific point in time, SCN, or log sequence.

### Example: Recover until time
```bash
RMAN> SHUTDOWN IMMEDIATE;
RMAN> STARTUP MOUNT;
RMAN> RUN {
  SET UNTIL TIME '2025-05-16 12:00:00';
  RESTORE DATABASE;
  RECOVER DATABASE;
}
RMAN> ALTER DATABASE OPEN RESETLOGS;
```
Use RESETLOGS after PITR to reset redo log sequence.

## Flashback Technology
Flashback provides a fast alternative to PITR:

### Flashback Database
```sql
FLASHBACK DATABASE TO TIMESTAMP TO_TIMESTAMP('2025-05-16 10:00:00', 'YYYY-MM-DD HH24:MI:SS');
```
Requires Flashback to be enabled and flashback logs configured.

### Common Backup and Recovery Errors
| Error             | Cause                   | Solution                      |
| ----------------- | ----------------------- | ----------------------------- |
| `ORA-01578`       | Block corruption        | Restore & recover datafile    |
| `ORA-03113`       | Connection lost         | Check instance status         |
| `RMAN-06054`      | No backup found         | Check backup catalog          |
| `ORA-01113/01110` | Datafile needs recovery | Use RMAN to recover           |
| `ORA-19809`       | No space for archivelog | Backup or delete archive logs |


### Best Practices
- Use RMAN regularly.
- Keep multiple backup copies (onsite/offsite).
- Enable ARCHIVELOG and Flashback.
- Schedule daily full and incremental backups.
- Test restore procedures periodically.
- Backup control file and SPFILE.
- Monitor archive log space (V$ARCHIVED_LOG, V$RECOVERY_FILE_DEST).

### Useful RMAN Commands Summary
```sql
# Backup full DB
BACKUP DATABASE;

# Backup database + archived logs
BACKUP DATABASE PLUS ARCHIVELOG;

# Restore and recover DB
RESTORE DATABASE;
RECOVER DATABASE;

# Backup control file
BACKUP CURRENT CONTROLFILE;

# Check backup info
LIST BACKUP;
REPORT OBSOLETE;

# Validate backups
VALIDATE BACKUPSET;

# Configure retention
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
```