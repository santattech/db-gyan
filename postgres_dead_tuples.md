## What is dead tuples?

In PostgreSQL, when a row is deleted or updated, PostgreSQL creates so-called Dead tuples. Dead tuples are not referenced in the current version of our databases' tables, but still occupy space on disk. This behavior supports PostgreSQL's multi-version concurrency control (MVCC), which allows transactions to run concurrently without interfering with each other. 

### How to check table size

SQL to execute:

```sql
SELECT table_name,pg_relation_size(table_schema || '.' || table_name) as size FROM information_schema.tables 
WHERE table_schema NOT IN ('information_schema', 'pg_catalog') ORDER BY size DESC;
```

OUTPUT:

![image](https://github.com/santattech/db-gyan/assets/1919373/712e7d5c-3991-468a-8189-49814d662383)

Note:
_If the name of the table has upcased letter, the above query will throw error, as we have used table_schema.table_name which relation will be actually missing in the SCHEMA._

## What is VACUUM

In PostgreSQL, `VACUUM` is a maintenance operation used to clean up and optimize the database by reclaiming storage occupied by dead tuples. When rows in a PostgreSQL table are updated or deleted, the old versions of the rows are not immediately removed; they remain in the table until a `VACUUM` operation is performed.  **VACUUM allows for database operations to continue unimpeded, while VACUUM FULL locks tables, preventing access until the operation is complete**.

### Usage and Examples

- Basic VACUUM - Reclaims space from dead tuples
- VACUUM FULL - Rewrite the tables to compact it.
- VACUMM Analyze - Reclaims space and updates statistics.
- AUTOVACUMM - PostgreSQL has an autovacuum daemon that runs in the background and automatically performs `VACUUM` operations on tables as needed. 

### How to run VACUUM

To run auto vacuum in PostgreSQL, you typically don't have to do anything special. PostgreSQL has a built-in autovacuum process that runs automatically to reclaim storage occupied by dead tuples and update statistics used by the query planner. However, you may need to tune some parameters in your postgresql.conf file to optimize autovacuum's behavior according to your database workload.

Autovacuum does not delete data from tables. Instead, it marks dead tuples (rows that are no longer visible to any transaction) as reusable space within the table. The actual space is reclaimed when new data is inserted or updated in the table. This process helps to prevent database bloat and ensures that your tables remain efficient.

If you want to manually trigger a vacuum operation on a specific table, you can use the VACUUM command. For example:

```sql
VACUUM FULL my_table;
```

The VACUUM FULL variant will also compact the table by rewriting it, potentially reclaiming more space but requiring exclusive locks on the table for the duration of the operation.

Just be cautious with manual vacuum operations, especially VACUUM FULL, as they can be resource-intensive and may impact the performance of your database, especially on large tables.

### Summary and AWS RDS behaviour

**In summary**, autovacuum is a background process that helps maintain the health and performance of your PostgreSQL database by reclaiming space and updating statistics automatically. It does not delete data but rather marks it for reuse. 
In **AWS RDS** , it generally executes, AUTOVACUUM, it keeps monitoring the tables and RDS maintains a threshold for auto vaccuming, If it reaches the threshold for a particular table, it performs the vacuum job. For your information, for now, you can keep in mind 4 columns or information which actually plays the role for determining the threashold.

1. `autovacuum_vacuum_threshold` 
2. `autovacuum_analyze_threshold` 
3. `autovacuum_vacuum_scale_factor`
4.  `autovacuum_analyze_scale_factor`

### Example

We have a table where we deleted huge data from a particular table `my_table`. Now the table contains around 80k records and while we check the size of the table, it shows the result in bytes and when we converted it to GB, it returns around 16 GB.  Which looks odd and then 

we ran the command `VACUUM FULL my_table`. It took around 2 minutes to complete the execution. And we check again for the table size occupied. Now it becomes 1 GB. So we almost recovered 15 GB from the disk-space.

### Notes

`VACUUM` can impact the performance while it runs, specially `VACUMM FULL` will lock the tables in the entire operation time.  It's often best to rely on the autovacuum daemon, which is designed to run `VACUUM` operations during periods of low database activity to minimize impact. Regular vacuuming helps maintain database performance and prevent excessive table bloat, which can slow down queries and increase storage requirements.

