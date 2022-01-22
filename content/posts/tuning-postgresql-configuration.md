---
title: "Tuning PostgreSQL Configuration"
date: 2018-11-02T11:16:48+02:00
categories:
- tech
tags:
- postgresql
---

## Configuration File

1. RAM amounts defined below doesn't represent the total amount of memory
   available for your machine, instead, it represents what PostgreSQL can use
   maximum.
2. Decribed configuration requires PostgreSQL >= 10. Remove `max_parallel_workers`
   for older versions.

The configuration file is usually located at `/etc/postgresql/$VERSION/main/postgresql.conf`
in Debian-derivatives. For locating the file in other operating systems, you can
use `psql` as follows:

```bash
psql -U postgres
show config_file;
```

## Tracking Query Statistics

Enable the `pg_stat` extension, if you are interested in tracking query statistics:

```bash
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
```

Then restart postgres:

```bash
service postgresql restart
```

## Sample Configuration

```conf
# Postgresql Version: 10 || 11
# OS Type: linux
# Total Memory (RAM): 8 GB
# CPUs num: 2
# Connections num: 200
# Data Storage: ssd

max_connections = 200
shared_buffers = 2GB
effective_cache_size = 6GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.7
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 11MB
min_wal_size = 1GB
max_wal_size = 2GB
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
```

## References

- <https://pgtune.leopard.in.ua/#/>
