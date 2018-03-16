title: Why PostgreSQL's ON CONFLICT Cannot Find My Partial Unique Index?
date: 2018-03-16 17:05:03
tags:
- PostgreSQL
---
## TL;DR

When using column names to infer a partial index with `ON CONFLICT()`, the exact predicates (`WHERE ...`) from the index must be added to the `ON CONFLICT` clause, which looks like:

```sql
INSERT INTO ...
  ON CONFLICT (column_1, column_2) WHERE column_2 > 0
    DO ...
```

- - -

PostgreSQL added support for UPSERT queries in version 9.5. An UPSERT query does the trick as an **atomic operation** that, if the record already exists in the target table, it will be updated with the new values, otherwise a new record will be inserted.

The way PostgreSQL implements UPSERT is that, instead of adding a new `UPSERT` method, it adds a new `ON CONFLICT` clause to INSERT queries. For example, if you have a `person` table which has some columns in it:

```sql
DROP TABLE IF EXISTS user;
CREATE TABLE person (
    id BIGSERIAL,
    company_id BIGINT NOT NULL, -- ID of company the user belongs to
    personnel_no BIGINT NOT NULL, -- an internal number for employees
    name VARCHAR(32) NOT NULL,
    PRIMARY KEY (id)
);
```

And you creates a unique index for `company_id` and `personnel_no`, as two employees of a same company cannot share one personnel number.

```sql
CREATE UNIQUE INDEX uniq_idx_company_personnel
  ON person(company_id, personnel_no);
```

However, not all people belong to a company. For those who don't have a job, the `company_id` is set to 0, and the unique index does not count for them. So you should turn it into a partial index like:

```sql
CREATE UNIQUE INDEX uniq_idx_company_personnel
  ON person(company_id, personnel_no) WHERE company_id > 0;
```

Now you want to add some people into this shiny new table. You choose to use UPSERT to make INSERT and UPDATE into one single query.

```sql
INSERT INTO person (company_id, personnel_no, name)
  VALUES (1, 1, "Boss")
  ON CONFLICT (company_id, personnel_no)
    DO UPDATE SET name = EXCLUDED.name;
```

Here, in the parentheses after `ON CONFLICT` are the columns corresponding to those in the unique index. `DO UPDATE SET name = EXCLUDED.name` means if there is conflict, update the existing record with the new name provided (which is "Boss"). `EXCLUDED` represents the record you are going to insert.

This is when PostgreSQL throws at you an error saying that:

```
ERROR:  there is no unique or exclusion constraint matching the ON CONFLICT specification
```

PostgreSQL cannot find your unique index based on the two columns `company_id` and `personnel_no`, even if the index does exist. Why?

Answer can be found in the [document](https://www.postgresql.org/docs/9.6/static/sql-insert.html#SQL-ON-CONFLICT) of INSERT query, which says:

> All table_name unique indexes that, without regard to order, contain exactly the conflict_target-specified columns/expressions are inferred (chosen) as arbiter indexes. If an index_predicate is specified, it must, as a further requirement for inference, satisfy arbiter indexes.

That means, if your unique index is a partial one, the predicates you added to `CREATE INDEX` must be all provided here, or the partial index will not be inferred.

So here's the solution:

```sql
INSERT INTO person (company_id, personnel_no, name)
  VALUES (1, 1, "Boss")
  ON CONFLICT (company_id, personnel_no) WHERE company_id > 0
    DO UPDATE SET name = EXCLUDED.name;
```
