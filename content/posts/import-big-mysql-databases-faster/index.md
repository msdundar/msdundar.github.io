---
title: "Import Big MySQL Databases Faster"
description: "Configuring InnoDB for faster MySQL imports."
date: 2017-04-05T11:44:19+02:00
categories:
- tech
tags:
- mysql
cover:
    image: "posts/import-big-mysql-databases-faster/assets/le-chateau-de-virieu-sur-bourbre-isere-1877-johan-barthold-jongkind.webp"
    alt: "Le Chateau De Virieu-Sur-Bourbre, Isere (1877) - Johan Barthold Jongkind"
    relative: false
images: ["assets/le-chateau-de-virieu-sur-bourbre-isere-1877-johan-barthold-jongkind.webp"]
---

I recently had to import some quite large SQL dumps to my local machine for data
analysis purposes. There were 10 different `.sql` dumps in total, and each of
them were sized more than 100GB.

## Naive Attempt

First of all, I tried to import each by using the `<` operator:

```sql
mysql some_database < some_dump.sql
```

Unsurprisingly, each task took very long to finish, approximately 6-7 hours
per dump file, in a brand new laptop with i7 CPU, 16GB RAM and 1TB SSD.
Since I didn't have 60 hours for importing all, I had to take a look for
solutions that can potentially speed up the process.

## Configuration

After Googling and reading through a set of similar issues on Stackoverflow, I
came across an [answer](https://dba.stackexchange.com/a/83385/166635),
referencing to [Vadim Tkachenko](https://twitter.com/vadimtk) and  explaining
why it was taking so long to import.

After grasping the reasons behind, I changed the `InnoDB` settings (`my.ini`)
as follows:

```
innodb_buffer_pool_size = 12G # 60% - 70% of your RAM size
```

```
innodb_log_buffer_size = 16M # 16M or 32M is fine
```

```
innodb_log_file_size = 3G # 25% of buffer pool size
```

```
innodb_write_io_threads = 32 # 32 is fine, 64 is maximum
```

```
innodb_flush_log_at_trx_commit = 0
```

That was it after restarting MySQL:

```bash
$ sudo service mysql restart
```

## Results

New settings were clearly effective on the import time of dump files. Each
import task started to take around 20-30 minutes after fine tuning the settings.

Cheers.
