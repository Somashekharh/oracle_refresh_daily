Oracle 23ai – Key Implementable Features After Upgrading from 19c (Single Instance)

1. JSON Relational Duality
- One data set, two access patterns – relational and JSON, no duplication.
- Define a JSON view over relational tables. Reads/writes can be done via JSON, but the underlying data stays in normalized tables.
- Example:
  CREATE JSON DUALITY VIEW customer_dv AS
  SELECT JSON{'cust_id' : c.id, 'name' : c.name, 'orders' : 
    (SELECT JSON{'order_id' : o.id, 'total' : o.amount} FROM orders o WHERE o.cid = c.id)}
  FROM customers c;
- Benefit: REST APIs / MongoDB-compatible access without schema redesign.

2. Operational Property Graphs (SQL/PGQ)
- Create property graphs directly over relational tables. Query using GRAPH_TABLE operator.
- Example:
  CREATE PROPERTY GRAPH bank_graph
    VERTEX TABLES (customers KEY(id), accounts KEY(id))
    EDGE TABLES (transfers SOURCE src_id DESTINATION dst_id);
  SELECT * FROM GRAPH_TABLE(bank_graph, 'MATCH (a)-[e]->(b) RETURN a.name, b.name');
- Use for fraud detection, recommendations, BOM analysis – native in DB.

3. SQL Enhancements You Can Use Immediately
- Boolean Data Type: BOOLEAN column, variable, expressions. SELECT * FROM tasks WHERE is_complete = TRUE;
- IF [NOT] EXISTS for DDL: DROP TABLE IF EXISTS temp_data; CREATE TABLE IF NOT EXISTS staging (...);
- Direct Joins in UPDATE/DELETE (simpler syntax, UPDATE ... FROM).
- Schema-Level Privileges: GRANT SELECT ANY TABLE ON SCHEMA hr TO analyst;
- Annotations: CREATE TABLE emp (id NUMBER ANNOTATIONS (HIDDEN 'YES'), ...); – metadata for tools.

4. Performance: Automatic Materialized Views & Faster Analytics
- Auto MVs / Real-Time MVs: REFRESH FAST ON STATEMENT ensures fresh data on query.
  CREATE MATERIALIZED VIEW sales_sum_mv REFRESH FAST ON STATEMENT AS SELECT ...;
- In-Memory Base Level: free 16 GB column store with 23ai. Enable with INMEMORY_SIZE, mark tables INMEMORY.
- SQL Firewall: DBMS_SQL_FIREWALL captures allow-lists and blocks unauthorized SQL patterns.

5. Developer Productivity
- JavaScript Stored Procedures (Multilingual Engine): 
  CREATE FUNCTION my_js RETURN NUMBER AS MLE LANGUAGE JAVASCRIPT 'return 42;';
- Microservices with ORDS: auto-generate REST APIs for tables/duality views.
- Schema Evolution for JSON: Data Guide, multi-value indexes.
- Lock-Free Reservable Columns: mark column RESERVABLE for booking scenarios, no row locks.

6. High Availability & Data Protection
- Rolling Application Upgrades with PDBs (Multitenant) – create and switch to updated PDB.
- Flashback Time Travel (SQL:2023 temporal tables):
  CREATE TABLE t (...) AS PERIOD FOR valid_time (...);
  SELECT * FROM t FOR valid_time AS OF TIMESTAMP '...';
- Blockchain/Immutable Tables: rows cannot be deleted/updated, cryptographically chained.
  CREATE BLOCKCHAIN TABLE ledger (...) NO DROP UNTIL 90 DAYS IDLE;

7. Security Hardening
- TDE improvements: simpler key rotation, auto-login keystores.
- Unified Audit Policies: CREATE AUDIT POLICY ... always on.
- Passwordless Schema Authentication: OCI IAM tokens, certificate-based.

Implementation Checklist (19c -> 23ai Single Instance):
1. Enable In-Memory Base Level (INMEMORY_SIZE=16G) and mark key tables INMEMORY.
2. Use BOOLEAN columns in new tables.
3. Add IF [NOT] EXISTS to scripts for idempotency.
4. Build JSON Duality Views over transactional data, expose via ORDS REST.
5. Replace MV refresh jobs with REFRESH FAST ON STATEMENT for real-time summarisation.
6. Create a SQL Firewall allow-list for critical service accounts.
7. Create a Blockchain table for immutable audit trails.
8. Apply Property Graphs to one analytical case (e.g., customer 360).
9. Use Annotations for table/column documentation.
10. Audit critical operations with Unified Audit.

Note: 23ai is the direct long-term release after 19c. Use AutoUpgrade to move your 19c non-CDB to a 23ai PDB.
