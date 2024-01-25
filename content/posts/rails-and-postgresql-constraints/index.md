---
title: "Rails and PostgreSQL: Constraints"
description: "Using PostgreSQL constraints with Ruby on Rails."
date: 2018-11-03T15:24:20+02:00
categories:
- tech
tags:
- rails
- postgresql
cover:
    image: "posts/rails-and-postgresql-constraints/assets/le-treport-le-matin-normandie-1852-johan-barthold-jongkind.webp"
    alt: "Le Treport, Le Matin, Normandie (1852) - Johan Barthold Jongkind"
    relative: false
images: ["assets/le-treport-le-matin-normandie-1852-johan-barthold-jongkind.webp"]
---

During [nokul](https://github.com/omu/nokul) we've heavily implemented PostgreSQL features into our
Rails application. Unfortunately, ActiveRecord doesn't come with constraint support for PostgreSQL,
but [rein](https://github.com/nullobject/rein) does a fantastic job covering what's missing in ActiveRecord.
We believe that, one shouldn't rely on a web application, that is very prone for human-error, when it comes
to data integrity. Therefore our PostgreSQL tables included various constraints and limits.

Below you will find a set of rules that we've investigated, implemented and battle tested with various types.

## Not Null Constraint & Presence Constraint

Use `null: false` for `foreign_key` columns, if there is no `optional: true` relation in between.

  ```ruby
  t.references :unit,
    null: false,
    foreign_key: true
  ```

Do not use `null: false` for `integer`, `boolean` and `float` types, instead use `add_null_constraint`:

  ```ruby
  add_null_constraint :students, :active
  ```

Do not use `null: false` for `string` type, instead use `add_presence_constraint`:

  ```ruby
  add_presence_constraint :countries, :name
  ```

--------

### Not Null Constraint vs. Not Null Check

It's possible to define `NOT NULL` case as a `CONSTRAINT` or as a `CHECK`, in PostgreSQL. However, there are
some differences between them:

- `change_column_null` (or `null: false`) adds a `CONSTRAINT` on a column,
- `add_presence_constraint` adds a `CHECK` to the table.

--------

### `change_column_null (null: false)`

`change_column_null` method of Rails adds a `not null` CONSTRAINT to column:

```ruby
change_column_null :cities, :country_id, false
```

```ruby
                                       Table "public.cities"
    Column    |          Type          | Collation | Nullable |              Default
--------------+------------------------+-----------+----------+------------------------------------
 id           | bigint                 |           | not null | nextval('cities_id_seq'::regclass)
 name         | character varying(255) |           | not null |
 country_id   | bigint                 |           | not null |

Indexes:
    "cities_pkey" PRIMARY KEY, btree (id)
    "cities_name_unique" UNIQUE CONSTRAINT, btree (name) DEFERRABLE
    "index_cities_on_country_id" btree (country_id)
```

--------

### `add_null_constraint`

`add_null_constraint` method of [rein](https://github.com/nullobject/rein) adds a `not_null` `CHECK` to table:

```ruby
add_null_constraint :cities, :country_id
```

```ruby
                                       Table "public.cities"
    Column    |          Type          | Collation | Nullable |              Default
--------------+------------------------+-----------+----------+------------------------------------
 id           | bigint                 |           | not null | nextval('cities_id_seq'::regclass)
 name         | character varying(255) |           | not null |
 country_id   | bigint                 |           |          |

Indexes:
    "cities_pkey" PRIMARY KEY, btree (id)
    "cities_name_unique" UNIQUE CONSTRAINT, btree (name) DEFERRABLE
    "index_cities_on_country_id" btree (country_id)
Check constraints:
    "cities_country_id_null" CHECK (country_id IS NOT NULL)
```

[PostgreSQL documentation](https://www.postgresql.org/docs/current/ddl-constraints.html#id-1.5.4.5.6) explains how `NOT NULL CHECK` and `NOT NULL CONSTRAINT` are similar, but `NOT NULL CONSTRAINT` is faster:

> A not-null constraint is always written as a column constraint. A not-null constraint is functionally equivalent to creating a check constraint CHECK (column_name IS NOT NULL), but in PostgreSQL creating an explicit not-null constraint is more efficient.

Unfortunately the official PostgreSQL documentation doesn't tell the performance difference in numbers. However, a [Stack Overflow user](https://dba.stackexchange.com/a/158644) mentions the performance difference as quite insignificant, around 0.5%.

--------

### `add_presence_constraint`

`add_presence_constraint` is used to check the existence of strings. It doesn't allow empty strings as
`add_null_constraint` does:

```ruby
User.create(email: ' ') # can't be created when add_presence_constraint is in place
```

--------

### Why `CHECK` instead of `CONSTRAINT`?

While the official PostgreSQL documentation mentions the performance difference between `CHECK` and `CONSTRAINT` in favor of `CONSTRAINT`, [rein](https://github.com/nullobject/rein) still adds a `CHECK` to satisfy `NOT NULL` condition, simply because of two reasons:

1. Reverting a `CHECK` is easy, but a whole column needs to be rewritten when reverting a `CONSTRAINT`.
2. Since a whole column needs to be re-written when there is a change in `CONSTRAINT`, an `AccessExclusiveLock`
  added to the table. On the other hand, changes on `CHECK` doesn't add an `AccessExclusiveLock` to tables,
  so that they don't cause down times.

--------

## Unique Constraint

`unique_constraint` works a little bit different than the others, it adds an index:

```ruby
users_email_unique
users_id_number_unique
```

If you want a `unique_constraint` to work as the latest step of a transaction (for performance reasons),
you can also `defer` it:

```ruby
add_unique_constraint :books, :isbn, deferred: true
```

--------

## References

- <https://www.postgresql.org/docs/current/ddl-constraints.html#id-1.5.4.5.6>
- <https://dba.stackexchange.com/a/158644>

