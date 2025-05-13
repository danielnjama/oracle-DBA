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

## Table of Contents

1. [Overview of Oracle Process Architecture](#overview-of-oracle-process-architecture)
2. [Types of Processes](#types-of-processes)
3. [Database Writer (DBWn)](#database-writer-dbwn)
4. [Log Writer (LGWR)](#log-writer-lgwr)
5. [Checkpoint (CKPT)](#checkpoint-ckpt)
6. [System Monitor (SMON)](#system-monitor-smon)
7. [Process Monitor (PMON)](#process-monitor-pmon)
8. [Recoverer Process (RECO)](#recoverer-process-reco)
9. [Archiver Process (ARCn)](#archiver-process-arcn)
10. [Other Background Processes](#other-background-processes)
11. [Conclusion](#conclusion)

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

1. [Overview of Memory Architecture](#overview-of-memory-architecture)
2. [System Global Area (SGA)](#system-global-area-sga)
   - [Database Buffer Cache](#database-buffer-cache)
   - [Shared Pool](#shared-pool)
   - [Redo Log Buffer](#redo-log-buffer)
   - [Large Pool](#large-pool)
   - [Java Pool & Streams Pool](#java-pool--streams-pool)
   - [Keep Buffer Pool](#keep-buffer-pool)
   - [Recycle Buffer Pool](#recycle-buffer-pool)
   - [Non-Default (nK) Buffer Cache](#non-default-nk-buffer-cache)
3. [Program Global Area (PGA)](#program-global-area-pga)
4. [Dedicated vs Shared Server Architecture](#dedicated-vs-shared-server-architecture)
5. [PGA in Shared Server Environment](#pga-in-shared-server-environment)
6. [Conclusion](#conclusion)

---

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


