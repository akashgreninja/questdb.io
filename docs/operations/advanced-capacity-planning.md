---
title: Advanced capacity planning
description:
  How to plan and configure system resources, database configuration, and client
  application code available to QuestDB to ensure that server operation
  continues uninterrupted.
---

Advanced capacity planning should be considered as part of the QuestDB
deployment to further optimize the instance performance. This page is aimed at
users who have been using QuestDB and are familiar with their data sets and who
are seeking to fine-tune the performance.

Most of the configuration settings referred to below are configured in QuestDB
by either a `server.conf` configuration file or as environment variables. For
more details on applying configuration settings in QuestDB, refer to the
[configuration](/docs/reference/configuration) page.

To monitor various metrics of the QuestDB instances, refer to the
[Prometheus monitoring page](/docs/third-party-tools/prometheus/) or the
[Health monitoring page](/docs/operations/health-monitoring/).

Refer to [Capacity planning](/docs/operations/capacity-planning/) for more
general resource planning.

## Storage and filesystem

The following sections describe aspects to be considered regarding the storage
of data and filesystem considerations.

### Symbol caching

[Symbol cache](/docs/concept/symbol#usage-of-symbols) enables the use of on-heap
cache for reads and can enhance performance. However, the cache size grows as
the number of distinct value increases, and the size of the cached symbol may
hinder query performance.

We recommend that users check the JVM and GC metrics via
[Prometheus monitoring](/docs/third-party-tools/prometheus/) before taking one
of the following steps:

- Disabling the symbol cache. See
  [Usage of `symbols`](/docs/concept/symbol#usage-of-symbols) for server-wide
  and table-wide configuration options.
- Increasing the JVM heap size using the `-Xmx` argument.

### Symbol capacity

[Symbol capacity](/docs/concept/symbol#usage-of-symbols) should be the same or
slightly larger than the count of distinct symbol values.

Undersized symbol columns slow down query performance. Similarly, there is a
performance impact when symbol is not used for its designed way, most commonly
assigning `symbol` to columns with too many distinct values. It is crucial to
choose a suitable [data type](/docs/reference/sql/datatypes/) based on the
nature of the dataset.

### Index

Appropriate us of [indexes](/docs/concept/indexes/) provides faster read access
to a table. However, indexes have a noticeable cost in terms of disk space and
ingestion rate - we recommend starting with no indexes and adding them later,
only if they appear to improve query performance. Refer to
[Index trade-offs](/docs/concept/indexes#trade-offs) for more information.

## CPU configuration

This section describes configuration strategies based on the forecast behavior
of the database.

### RAM size

We recommend having at least 8GB of RAM for basic workloads and 32GB for more
advanced ones.

For relatively small datasets, typically a few to a few dozen GB, if the need
for reads is high, performance can benefit from maximizing the use of the OS
page cache. Users may consider increasing available RAM to improve the speed of
read operations.

### Write amplification

When ingesting out-of-order data, high disk write rate combined with high write
amplification may slow down the performance.

For data ingestion over InfluxDB Line Protocol (ILP), the first step to resolve
this start from adjusting the
[commit lag](/docs/guides/out-of-order-commit-lag/) value and the
`cairo.max.uncommitted.rows` value.

For data ingestion over PGWire, or as a further step for ILP ingestion, smaller
table [partitions](/docs/concept/partitions/) maybe reduce the write
amplification. This applies to tables with partition directories exceeding a few
hundred MBs on disk. For example, partition by day can be reduced to by hour,
partition by month to by day, and so on.

:::note

- For more information on commit lag, please refer to the
  [ILP commit strategy page](/docs/reference/api/ilp/tcp-receiver/#commit-strategy).
- In QuestDB the write amplification is calculated by the
  [metrics](/docs/third-party-tools/prometheus#scraping-prometheus-metrics-from-questdb)
  `questdb_physically_written_rows_total` / `questdb_committed_rows_total`.
- Partitions are defined when a table is created. Refer to
  [CREATE TABLE](/docs/reference/sql/create-table/) for more information.

:::

### Memory page size configuration

For frequent out-of-order (O3) writes over high number of columns/tables, the
performance may be impacted by the size of the memory page being too big as this
increases the demand for RAM.16M by default. The memory page,
`cairo.o3.column.memory.size`, is set to 16M by default. Decreasing the value in
the interval of [128K, 8M] based on the number of columns used may improve O3
write performance.
