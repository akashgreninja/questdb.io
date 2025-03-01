---
title: FAQ
description: FAQ for QuestDB troubleshooting.
---

The following document contains common hardware and software configuration
issues met when running QuestDB, as well as solutions to them.

## Why is ILP data not immediately available?

InfluxDB line protocol (ILP) does not commit data on single lines or when the
sender disconnects, but instead uses a number of rules to break incoming data
into commit batches. This results in data not being visible in `SELECT` queries
immediately after being received. Refer to our
[guide to commit lag](/docs/guides/out-of-order-commit-lag) to understand the
concept and
[InfluxDB line protocol reference](/docs/reference/api/ilp/tcp-receiver#commit-strategy)
to understand these rules.

## How do I delete a row?

See our guide on [modifying data](/docs/guides/modifying-data).

## How do I convert a `STRING` column to a `SYMBOL` or vice versa?

The SQL `UPDATE` keyword can be used to change the data type of a column. The
same approach can also be used to increase the
[capacity of a `SYMBOL` column](/docs/concept/symbol#symbol-columns) that is
undersized.

The steps are as follows:

1. Add a new column to the table and define the desired data type.
1. Stop data ingestion and increase
   [SQL query timeout](/docs/reference/configuration/#cairo-engine/),
   `query.timeout.sec`, as `UPDATE` may take a while to complete. Depending on
   the size of the column, we recommend to increase the value significantly: the
   default is 60 seconds and it may be reasonable to increase it to one hour.
   Restart the instance after changing the configuration, to activate the
   change.
1. Use `UPDATE` to copy the existing column content to the new column. Now, the
   column has the correct content with the new data type.
1. Delete the old column.
1. Rename the new column accordingly.
For example, to change `old_col` from `STRING` to `SYMBOL` for table `my_table`:

```questdb-sql
ALTER TABLE my_table ADD COLUMN new_col SYMBOL;
UPDATE my_table SET new_col = old_col;
ALTER TABLE my_table DROP COLUMN old_col;
ALTER TABLE my_table RENAME COLUMN new_col TO old_col;
```

## Why do I get `table busy` error messages when inserting data over PostgreSQL wire protocol?

You may get `table busy [reason=insert]` or similar errors when running `INSERT`
statements concurrently on the same table. This means that the table is locked
by inserts issued from another SQL connection or other client protocols for data
import, like ILP over TCP or CSV over HTTP. To reduce the chances of getting
this error, try using auto-commit to keep the transaction as short as possible.

We're also considering adding automatic insert retries on the database side, but
for now, it is safe to handle this error on the client side and retry the
insert.

## Why do I see `could not open read-write` messages when creating a table or inserting rows?

Log messages may appear like the following:

```
2022-02-01T13:40:11.336011Z I i.q.c.l.t.LineTcpMeasurementScheduler could not create table [tableName=cpu, ex=could not open read-write
...
io.questdb.cairo.CairoException: [24] could not open read-only [file=/root/.questdb/db/cpu/service.k]
```

The machine may have insufficient limits for the maximum number of open files.
Try checking the `ulimit` value on your machine. Refer to
[capacity planning](/docs/operations/capacity-planning#maximum-open-files) page
for more details.

## Why do I see `errno=12` mmap messages in the server logs?

Log messages may appear like the following:

```
2022-02-01T13:40:10.636014Z E i.q.c.l.t.LineTcpConnectionContext [8655] could not process line data [table=test_table, msg=could not mmap  [size=248, offset=0, fd=1766, memUsed=314809894008, fileLen=8192], errno=12]
```

The machine may have insufficient limits of memory map areas a process may have.
Try checking and increasing the `vm.max_map_count` value on your machine. Refer
to
[capacity planning](/docs/operations/capacity-planning#max-virtual-memory-areas-limit)
page for more details.

## How do I avoid duplicate rows with identical fields?

We have an open
[feature request to optionally de-duplicate rows](https://github.com/questdb/roadmap/issues/3)
inserted with identical fields. Until then, you need to
[modify the data](/docs/guides/modifying-data) after it's inserted and use a
`GROUP BY` query to identify duplicates.
