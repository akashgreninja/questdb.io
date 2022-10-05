---
title: Advanced capacity planning
description:
  How to plan and configure system resources available to QuestDB to ensure that
  server operation continues uninterrupted.
---

Advanced capacity planning should be considered as part of the QuestDB
deployment to further optimize the instance performance. This page is aimed at
users who have been using QuestDB and are familiar with their data sets and who
are seeking to fine-tune the performance. 

Most of the configuration settings referred to below are
configured in QuestDB by either a `server.conf` configuration file or as
environment variables. For more details on applying configuration settings in
QuestDB, refer to the [configuration](/docs/reference/configuration) page.

To monitor various metrics of the QuestDB instances, refer to the
[Prometheus monitoring page](/docs/third-party-tools/prometheus/) or the
[Health monitoring page](/docs/operations/health-monitoring/).

Refer to
[Capacity planning](/docs/operations/capacity-planning/) for more general
resource planning.


## Storage and filesystem

The following sections describe aspects to be considered regarding the storage
of data and filesystem considerations.

### Symbol caching

[Caching symbols](/docs/concept/symbol#usage-of-symbols) enables the use of on-heap cache for reads and can enhance performance. However, the cache size grows as the number of distinct value increases, and the size of the cached symbol may hinder query performance. 

We recommend that users check the JVM and GC metrics via [Prometheus monitoring](/docs/third-party-tools/prometheus/) before taking one of the following steps:

- Disabling symbol caching. See [Usage of `symbols`](/docs/concept/symbol#usage-of-symbols) for server-wide and table-wide configuration options.
- Increasing the JVM heap size using the `-Xmx` argument.

### Symbol capacity

[Symbol capacity](/docs/concept/symbol#usage-of-symbols) should be the same or slightly larger than the count of distinct symbol values.

Undersized symbol columns slows down query performance.
Similarly, there is a performance impact when symbol is not used for its designed way - assigning `symbol` to unique data. It is crucial to choose a suitable data type based on the nature of the dataset.

### Index

Indexes have a noticeable cost in terms of disk space and ingestion rate - we recommend starting with no indexes and adding them later, only if they appear to improve query performance.

## CPU configuration

### High disk write rate combined with high write amplification

### High RSS value (memory consumption) on write-most loads

### High disk read rate on a small dataset
