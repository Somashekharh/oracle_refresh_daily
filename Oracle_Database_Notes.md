# Oracle Database – Learning Notes
From beginner to advanced: architecture, SQL, administration, and tuning.

## 1. Introduction
- Oracle Database is a relational database management system (RDBMS) from Oracle Corporation.
- Supports multi-user, high-transaction environments, with strong consistency, security, and availability features.
- Editions: Express (XE, free), Standard, Enterprise (full features).
- Current version naming: Oracle 19c, 21c, 23ai (Long Term Release). 'c' = Cloud, 'i' = Internet, 'g' = Grid.

## 2. Architecture Overview
### Instance vs Database
- Database: physical files (data files, control files, redo logs).
- Instance: memory structures + background processes that access the database.
- Single database can have multiple instances (RAC). An instance mounts and opens a database.

### Memory Structures
- SGA (System Global Area): shared by all sessions.
  - Database Buffer Cache: caches data blocks.
  - Shared Pool: library cache (SQL, PL/SQL), dictionary cache.
  - Redo Log Buffer: redo entries before writing to disk.
  - Large Pool, Java Pool, Streams Pool (optional).
- PGA (Program Global Area): private memory per server process (sort area, session variables).
- In-memory Column Store (IM column store): optional in-memory area for columnar data.

### Background Processes
- SMON (System Monitor): instance recovery, space management.
- PMON (Process Monitor): cleans up failed user processes.
- DBWn (Database Writer): writes dirty buffers to datafiles.
- LGWR (Log Writer): writes redo from log buffer to online redo logs.
- CKPT (Checkpoint): updates datafile headers and control file.
- ARCn (Archiver): copies filled online redo logs to archive logs (in archivelog mode).
- Others: RECO (distributed recovery), CJQ0 (job coordinator), MMON/MMNL (AWR advisor).

### Real Application Clusters (RAC)
- Multiple instances on different servers share a single physical database.
- Uses Global Cache Service and additional processes (LMON, LMD, LMS, etc.) for cache fusion.

## 3. Storage Structures
### Physical
- Datafiles (.dbf): store actual data.
- Control files (.ctl): database structure info, checkpoint info, RMAN backups.
- Redo Log Files (.log): record all changes for recovery.
- Archived Redo Logs: copies of filled redo logs (when in ARCHIVELOG mode).
- Parameter file (spfile/pfile), password file, alert log, trace files.

### Logical
- Tablespace: logical storage unit, groups one or more datafiles.
- Segment: space allocated for a table, index, undo, etc.
- Extent: contiguous set of data blocks allocated to a segment.
- Block: smallest unit of storage (usually 8 KB).

### Important Tablespaces
- SYSTEM: data dictionary.
- SYSAUX: auxiliary for add-ons.
- UNDO (or UNDOTBS1): undo records.
- TEMP: temporary tablespace for sorts.
- USERS: default for user objects.

## 4. Networking
- Listener: server process listening for incoming client connections (default port 1521).
  Configuration: listener.ora file.
  Commands: lsnrctl start/stop/status.
- Client connection:
  - Easy Connect: sqlplus user/pass@host:port/service_name
  - TNS Names: tnsnames.ora maps a net service name to a connect descriptor.
- Service Name vs SID: SID identifies instance, Service Name identifies a workload (can be multiple for RAC).

## 5. SQL Fundamentals
### Query Basics
SELECT column1, column2 FROM table WHERE condition ORDER BY col;
- Use DISTINCT, AS alias, || for concatenation.

### DML (Data Manipulation)
INSERT INTO table (col1, col2) VALUES (val1, val2);
UPDATE table SET col = value WHERE condition;
DELETE FROM table WHERE condition;
MERGE: upsert logic.

### DDL (Data Definition)
CREATE TABLE t (id NUMBER PRIMARY KEY, name VARCHAR2(50));
ALTER TABLE t ADD (col DATE);
DROP TABLE t;
TRUNCATE TABLE t; -- faster, no rollback data
RENAME old_name TO new_name;
CREATE INDEX idx_name ON t(col);

### DCL (Data Control)
GRANT SELECT ON table TO user;
REVOKE SELECT ON table FROM user;
GRANT CREATE SESSION TO user; -- system privilege

### TCL (Transaction Control)
COMMIT;
ROLLBACK;
SAVEPOINT sp1; ROLLBACK TO sp1;

### Joins & Subqueries
- Inner Join, Left/Right Outer Join (old syntax: WHERE t1.col = t2.col(+))
- Subqueries: IN, EXISTS, NOT EXISTS, scalar subqueries.
- Set operators: UNION, UNION ALL, INTERSECT, MINUS.

### Built-in Functions
- String: UPPER, LOWER, SUBSTR, INSTR, LENGTH, TRIM, REPLACE.
- Numeric: ROUND, TRUNC, MOD, CEIL, FLOOR.
- Date: SYSDATE, TO_DATE, TO_CHAR, ADD_MONTHS, MONTHS_BETWEEN, NEXT_DAY.
- Conversion: TO_CHAR, TO_NUMBER, TO_DATE.
- NVL, NVL2, NULLIF, COALESCE, DECODE, CASE.

### Analytical Functions
ROW_NUMBER(), RANK(), DENSE_RANK(), LEAD(), LAG(), SUM() OVER (PARTITION BY … ORDER BY …).

## 6. PL/SQL Basics
- Block structure:
DECLARE
   v_var VARCHAR2(10);
BEGIN
   SELECT col INTO v_var FROM tab WHERE id=1;
   DBMS_OUTPUT.PUT_LINE(v_var);
EXCEPTION
   WHEN NO_DATA_FOUND THEN ...
END;
/
- Procedures, Functions, Packages.
- Cursors: implicit (SELECT INTO), explicit (OPEN, FETCH, CLOSE), cursor FOR loops.
- Exception handling: predefined (NO_DATA_FOUND, TOO_MANY_ROWS), user-defined.
- Triggers: BEFORE/AFTER INSERT/UPDATE/DELETE, INSTEAD OF.
- Collections: VARRAY, Nested Tables, Associative Arrays.
- Dynamic SQL: EXECUTE IMMEDIATE, DBMS_SQL.

## 7. Starting and Stopping the Database
### Startup Phases (using SQL*Plus as SYSDBA)
STARTUP NOMOUNT;  -- instance started, parameter file read, no database access
ALTER DATABASE MOUNT; -- control file read, database mounted
ALTER DATABASE OPEN;   -- datafiles and redo logs opened
- Or simply: STARTUP (does NOMOUNT -> MOUNT -> OPEN)

### Shutdown Modes
SHUTDOWN NORMAL (waits for all connections to end)
SHUTDOWN IMMEDIATE (rolls back active transactions, disconnects)
SHUTDOWN TRANSACTIONAL (waits for current transactions to finish)
SHUTDOWN ABORT (like crash, requires instance recovery next startup)

### Parameter File
- pfile (text, init.ora) or spfile (binary, server parameter file).
- Show parameters: SHOW PARAMETER parameter;
- Change dynamically: ALTER SYSTEM SET param=value SCOPE=MEMORY|SPFILE|BOTH;

## 8. Users, Security, and Privileges
- Create user: CREATE USER john IDENTIFIED BY passwd DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp QUOTA UNLIMITED ON users;
- Grant connect: GRANT CREATE SESSION, CREATE TABLE TO john;
- Roles: predefined (CONNECT, RESOURCE, DBA) or custom.
- Profiles: resource limits (password life, idle time, CPU per call).
  CREATE PROFILE app_user LIMIT FAILED_LOGIN_ATTEMPTS 5;
  ALTER USER john PROFILE app_user;
- Auditing: AUDIT SELECT ON hr.employees BY ACCESS;

## 9. Backup and Recovery
### RMAN (Recovery Manager)
- Main tool for backups.
- Connect: rman target /
- Backups:
  BACKUP DATABASE;
  BACKUP TABLESPACE users;
  BACKUP ARCHIVELOG ALL;
- Recovery:
  RESTORE DATABASE;
  RECOVER DATABASE;
- Full vs Incremental backups: INCREMENTAL LEVEL 0 (full), LEVEL 1 (differential/cumulative).
- Catalog database (optional) to store backup metadata.

### Archivelog Mode
- Required for hot backups and point-in-time recovery.
- Enable: SHUTDOWN IMMEDIATE; STARTUP MOUNT; ALTER DATABASE ARCHIVELOG; ALTER DATABASE OPEN;
- Archive destination: LOG_ARCHIVE_DEST_n parameters.

### Flashback Technology
- Flashback Query: SELECT ... AS OF TIMESTAMP ...
- Flashback Table: FLASHBACK TABLE tab TO TIMESTAMP ...
- Flashback Drop: FLASHBACK TABLE tab TO BEFORE DROP;
- Flashback Database: FLASHBACK DATABASE TO SCN ... (requires flashback logs, enabled via DB_FLASHBACK_RETENTION_TARGET)

## 10. Performance Tuning
### Understanding Execution Plans
EXPLAIN PLAN FOR SELECT ...;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
- Use DBMS_XPLAN.DISPLAY_CURSOR to see actual plan of a running query.
- Look for full table scans, high cost, cartesian joins.

### Indexes
- B-tree (default), Bitmap (low cardinality), Function-based, Reverse key.
- Monitor usage: ALTER INDEX idx MONITORING USAGE;
- Gather statistics: EXEC DBMS_STATS.GATHER_TABLE_STATS('schema','table');

### Automatic Workload Repository (AWR)
- Built-in performance data snapshots (default every 60 min).
- Reports: @?/rdbms/admin/awrrpt.sql (AWR report) or ADDM report.
- ADDM (Automatic Database Diagnostic Monitor) analyzes AWR data.

### SQL Tuning Advisor & SQL Access Advisor
- SQL Tuning Advisor: analyze a SQL statement and get recommendations.
  DECLARE task_name VARCHAR2(30);
  BEGIN task_name := DBMS_SQLTUNE.CREATE_TUNING_TASK(sql_id=>'...'); 
  DBMS_SQLTUNE.EXECUTE_TUNING_TASK(task_name); END;
- SQL Access Advisor: recommends indexes, materialized views.

### Memory Tuning
- Automatic Memory Management (MEMORY_TARGET), Automatic SGA Management (SGA_TARGET).
- Monitor with V$ views: V$SGASTAT, V$PGASTAT, V$MEMORY_TARGET_ADVICE.
- Use buffer cache advisory: V$DB_CACHE_ADVICE.

### Wait Events & Sessions
- V$SESSION_WAIT, V$SYSTEM_EVENT.
- Top wait events: db file sequential read (index reads), db file scattered read (full scan), log file sync (commit waits), enq: TX – row lock contention.

## 11. High Availability
### Data Guard
- Primary database ships redo to one or more standby databases (physical or logical).
- Modes: Maximum Protection, Maximum Availability, Maximum Performance.
- Role transitions: Switchover (planned), Failover (unplanned).
- Active Data Guard: physical standby open read-only while applying redo (requires license).

### Real Application Clusters (RAC)
- Multiple instances access the same database for scalability and high availability.
- Requires shared storage, clusterware (Grid Infrastructure), and special configuration.
- Connection load balancing via SCAN (Single Client Access Name) and services.

## 12. Data Movement Utilities
### Data Pump (expdp / impdp)
- High-speed export/import of tables, schemas, tablespaces, or entire database.
  expdp hr/hr DIRECTORY=dpump_dir DUMPFILE=hr.dmp SCHEMAS=hr
  impdp hr/hr DIRECTORY=dpump_dir DUMPFILE=hr.dmp REMAP_SCHEMA=hr:hr2
- Data Pump works via DBMS_DATAPUMP package or command-line clients.

### SQL*Loader
- Load data from flat files into tables.
  sqlldr userid=scott/tiger control=loader.ctl log=load.log
  Control file specifies data format, field delimiter, etc.

### External Tables
- Query flat files as if they were Oracle tables using CREATE TABLE ... ORGANIZATION EXTERNAL.

## 13. Monitoring and Logs
- Alert Log: database messages, errors, startup/shutdown, log switches. 
  Location: $(ADR_HOME)/alert/log.xml (text version in trace directory).
- DDL Log (optional): log schema changes.
- Dynamic Performance Views (V$ views): real-time info (V$SESSION, V$SQL, V$LOCK, V$INSTANCE).
- Enterprise Manager (EM Express or Cloud Control): web-based management.

Useful V$ views:
V$DATABASE, V$INSTANCE, V$VERSION, V$TABLESPACE, V$DATAFILE, V$LOGFILE, V$SESSION, V$LOCK, V$PROCESS, V$SQL, V$SQL_PLAN.

## 14. Advanced Topics (Brief)
- Partitioning: range, list, hash, composite. Improves performance and manageability.
- Materialized Views: pre-computed summary tables for fast querying; use query rewrite.
- Database Links: connection from one database to another.
  CREATE DATABASE LINK remote_db CONNECT TO user IDENTIFIED BY pwd USING 'tns_name';
- Scheduler: DBMS_SCHEDULER to run jobs, chains, and programs.
- Compression: table/index compression to save space and boost I/O.
- Multitenant (from 12c): Container Database (CDB) with pluggable databases (PDBs).
- In-Memory: column store in SGA for analytics.
- JSON, Graph, Blockchain tables (23ai features).

## 15. Useful Commands Cheat Sheet
Start SQL*Plus: sqlplus / as sysdba
Check database status: SELECT STATUS FROM V$INSTANCE;
List datafiles: SELECT NAME FROM V$DATAFILE;
List tablespaces: SELECT TABLESPACE_NAME FROM DBA_TABLESPACES;
Check archive log mode: ARCHIVE LOG LIST;
Switch logfile: ALTER SYSTEM SWITCH LOGFILE;
Force checkpoint: ALTER SYSTEM CHECKPOINT;
Kill session: ALTER SYSTEM KILL SESSION 'sid,serial#';
Find blocking locks: SELECT * FROM DBA_BLOCKERS;

## 16. Learning Path Recommendation
1. Understand Oracle architecture (instance, memory, storage).
2. SQL fundamentals, joins, grouping, subqueries.
3. PL/SQL programming basics.
4. DBA essentials: user management, backups, startup/shutdown, networking.
5. Performance tuning using execution plans and AWR.
6. RMAN backup and recovery.
7. High availability concepts (Data Guard, RAC).
8. Advanced administration (partitioning, multitenant, Data Pump).
