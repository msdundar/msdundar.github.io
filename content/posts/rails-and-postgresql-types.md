---
title: "Rails and PostgreSQL: Types"
date: 2018-11-09T19:24:20+02:00
categories:
- tech
tags:
- rails
- postgresql
---

During [nokul](https://github.com/omu/nokul) we've heavily implemented PostgreSQL features into our
Rails application. Unfortunately, ActiveRecord doesn't come with constraint support for PostgreSQL,
but [rein](https://github.com/nullobject/rein) does a fantastic job covering what's missing in ActiveRecord.
We believe that, one shouldn't rely on a web application, that is very prone for human-error, when it comes
to data integrity. Therefore our PostgreSQL tables included various constraints and limits.

Below you will find a set of rules that we've investigated, implemented and battle tested with various types.

--------

## String Type

- Do not use `limit: N` in migrations for `string` type. Instead, use `add_length_constraint`.

- If you aren't sure about the length of a string field, add `255` limit in the constraint, similar to the MySQL approach:

  ```ruby
  add_length_constraint :table, :column, less_than_or_equal_to: 255
  ```

- If length of an attribute is constant, consider using `equal_to`:

  ```ruby
  add_length_constraint :users, :id_number, equal_to: 11
  ```

- Do not use `text` type in migrations. Instead use `string` type and `add_length_constraint` with `65535`
  limit, similar to the MySQL approach. This will protect you from cells with crazy amount of data:

  ```ruby
  add_length_constraint :decisions, :description, less_than_or_equal_to: 65535
  ```

- Use `presence` and `unique` constraints whenever necessary:

  ```ruby
  add_presence_constraint :countries, :name
  add_unique_constraint :users, :email
  ```

--------

### `varchar` VS `varchar(n)` VS `char` VS `text`

- PostgreSQL doesn't have a limit for `string` type by default. Technically, one can insert gigabytes of data into a string field, therefore we've adopted a hard limit, `65535`, similar to the MySQL approach.
- There isn't a performance or size difference between `varchar (n)` and `text` in PostgreSQL.
- `varchar(n)` LOCKs the table in case of a change, that often causes downtimes. Therefore `limits` shouldn't be added to the column, instead they should be added as a `CHECK`.

--------

## Integer Type

- Do not use `limit: N` in migrations for `integer` type.

- Integers attributes often expected to return `0`, instead of `nil` in case of non-existence. Therefore,
  add a `null_constraint` if you don't have a good reason not to:

  ```ruby
  add_null_constraint :users, :articles_count
  ```

- Defining a default value (often `0`) will make sense most of the time. Also add a `null_constraint` together
  with the default value:

  ```ruby
  t.integer :articles_count, default: 0
  add_null_constraint :users, :articles_count
  ```

- Add a `numericality_constraint` constraint, if you aren't planning to accept a negative value in the column:

  ```ruby
  add_numericality_constraint :users, :articles_count,
                                      greater_than_or_equal_to: 0
  ```

- For numbers with exact upper and lower bounds, add `numericality_constraint`:

  ```ruby
  add_numericality_constraint :articles, :month,
                                          greater_than_or_equal_to: 1,
                                          less_than_or_equal_to: 12
  add_numericality_constraint :articles, :year,
                                          greater_than_or_equal_to: 1950,
                                          less_than_or_equal_to: 2050
  ```

--------

## Float Type

- `float`: Useful when accuracy isn't very important and when you're only interested in 3-5 numbers after the comma. Also useful when you are running complex arithmetic with these numbers.
- `decimal`: Useful when accuracy is very important (as in money), even more important than the performance.

- For both types, always add `null_constraint` if you've defined a `default` value:

  ```ruby
  t.decimal :min_credit, precision: 5, scale: 2, default: 0
  add_null_constraint :course_types, :min_credit
  ```

- For both types, add a `numericality_constraint` if you aren't accepting negative values:

  ```ruby
  add_numericality_constraint :course_types, :min_credit,
                                             greater_than_or_equal_to: 0
  ```

- Often a `float` or `decimal` attribute isn't expected to return `nil` in case of non-existence, instead, they're
  expected to return 0. Therefore, add a `null_constraint` if you don't have a good reason not to do so:

  ```ruby
  add_null_constraint :course_types, :min_credit
  ```

--------

### Float & Decimal

There are some differences between `float` and `decimal` in PostgreSQL. First of all, let's start with checking
what Rails produces for each type:

```ruby
t.float :incentive_point

| Column          | Type             | Nullable |
| --------------- | ---------------- | -------- |
| incentive_point | double precision |          |
```

```ruby
t.decimal :min_credit, precision: 5, scale: 2

| Column | Type         | Nullable |
| ------ | ------------ | -------- |
| credit | numeric(5,2) |          |
```

So, types in Rails converted into the following types in PostgreSQL:

- `float` -> `double_precision`
- `decimal` -> `numeric(x, y)` & `decimal(x, y)`

If we dig into these types in PostgreSQL:

```
| name             | size     | description | range                                                                       | in-rails |
| ---------------- | -------- | ----------- | --------------------------------------------------------------------------- | -------- |
| decimal (p, s)   | variable | exact       | p(total digits), s(digits after decimal point), max(p)=131072, max(s)=16383 | decimal  |
| numeric (p, s)   | variable | exact       | p(total digits), s(digits after decimal point), max(p)=131072, max(s)=16383 | decimal  |
| double-precision | 8-bytes  | inexact     | 15 significant digits, unlimited size                                       | float    |
```

Briefly:

- `decimal` and `numeric` types are the same in PostgreSQL.
- `decimal` and `numeric` types are **exact**, but `double-precision` is **inexact**.
- `decimal` and `numeric` types have some limits, while `double-precision` can be unlimited.

### Exact?

`exact` types store the data as it's submitted, however `inexact` types aren't. Therefore, `numberic` type should
be preferred in cases where precision is important.

--------

## Boolean Type

- Always add a `null_constraint` for `boolean` columns:

  ```ruby
  add_null_constraint :students, :active
  ```

- Always add a `default` value for `boolean` columns:

  ```ruby
  t.boolean :students, active: false
  ```

A `boolean` field often isn't expected to return `nil`. For example:

  ```ruby
  Student.where(active: nil).count => 50
  Student.where(active: false).count => 40
  Student.where(active: true).count => 60
  ```

In this case, it's very hard to tell how many students are active, and how many of them aren't. The data shown above needs to be corrected to before analyzed. Therefore, stick with the two rules explained here.

--------

## Reference Type

- Use `null: false` for `foreign_key` columns, if there is no `optional: true` relation in between:
- Add `foreign_key` constraint to ensure referential integrity:

  ```ruby
  t.references :unit,
    null: false,
    foreign_key: true
  ```

- Add `foreign_key` constraints directly to the column, instead of as a `CHECK`, as these columns typically not
  expected to change very often.

--------

## References

- <https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/>
- <https://stackoverflow.com/questions/4848964/postgresql-difference-between-text-and-varchar-character-varying>
- <https://dba.stackexchange.com/questions/125499/what-is-the-overhead-for-varcharn/125526#125526>
- <https://dba.stackexchange.com/questions/89429/would-index-lookup-be-noticeably-faster-with-char-vs-varchar-when-all-values-are/89433#89433>
- <https://gist.github.com/icyleaf/9089250>
- <https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html>
- <https://github.com/rails/rails/blob/master/activerecord/lib/active_record/connection_adapters/postgresql_adapter.rb>
- <https://stackoverflow.com/questions/38053596/benchmark-bigint-vs-int-on-postgresql>
- <https://stackoverflow.com/questions/2966524/calculating-and-saving-space-in-postgresql/7431468#7431468>
- <http://forums.devshed.com/postgresql-help-21/numeric-vs-float-607281.html>
- <https://stackoverflow.com/a/20887107/818033>
- <https://www.linuxtopia.org/online_books/database_guides/Practical_PostgreSQL_database/PostgreSQL_x2632_004.htm>
- <https://stackoverflow.com/a/8523253/818033>
