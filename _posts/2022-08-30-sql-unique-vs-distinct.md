---
author: borko
title: "SQL: NULL values in UNIQUE vs DISTINCT"
description: "There is a subtle difference in how SQL UNIQUE and DISTINCT reason about NULL values. For UNIQUE they are not the same, but for DISTINCT they are 🤯"
date: 2022-08-30 20:10:00 +0200
categories: [databases]
tags: [databases,sql]
image: /assets/img/posts/2022-08-30-sql-unique-vs-distinct/thumbnail.jpg
---

There is a subtle difference in how SQL `UNIQUE` and `DISTINCT` reason about `NULL` values. For `UNIQUE` they are not the same, but for `DISTINCT` they are 🤯

---

## Cheat Sheet

- **UNIQUE** constraint makes sure no values will be the same in that column except for `null` values.
- **DISTINCT** on the column will return all possible values (including `null`) exactly once.

---

## UNIQUE

Let's say you have a very simple table `customer` that has a primary key `id` that is also **auto-increment**, and two more columns: `name` and `personal_id`.

Furthermore, you know that `personal_id` cannot be the same for any of the customers, so we'll put the **UNIQUE** constraint on this column.

DDL for such a table (for **Postgresql**) would be like this:

```sql
create table customer (
    id Serial primary key,
    name varchar(50),
    personal_id varchar(50) unique
);
```
{: file='create_table_customer.sql'}

Now, let's populate this table with some data:

```sql
INSERT INTO customer(name, personal_id)
VALUES
('Darrel Beckett',      '875820185015303160')
('Franklyn Paget',      null)
('Russ Thompkins',      '173662838850888606')
('Jackson Martinson',   '459054575053219773')
('Yolanda Frost',       null)
('Lindsay Granger',     '625931342427148797')
('Corbin Steele',       '916300003568751325')
('Christen Hanson',     null)
```
{: file='insert-data.sql'}

We can notice there are multiple rows with `null` value for a `personal_id`.

The reasoning behind this is as follows:

> With a **UNIQUE** constraint `null` values are interpreted as **unknown**, therefore not breaking the rule of the uniqueness of the column.
> 
> In other words, a `null` is the `abscence of the value` rather than the value itself. It's simply a placeholder for a possible value, which can be filled in later on.
{: .prompt-info }

---

## **DISTINCT**

Now let's try to get distinct values from `personal_id` column in our table:

```sql
SELECT distinct personal_id
from customer;
```

What exactly **Distinct** is doing? As the name suggests, it returns distinct values, so any repeated value would be returned only once. Unlike for the **UNIQUE** constraint, from **DISTINCT** point of view, all `null` values are the same. The reasoning behind this is that there is at least one `unknown` value, rather than returning all `unknown` values.

The result should not be surprising:

```
personal_id
(null)
173662838850888606
875820185015303160
459054575053219773
916300003568751325
625931342427148797
```
{: file='output'}

What you **should** take care of is how you interpret those results.

> number of values in **UNIQUE** column **is not always the same** as number of values returned by **DISTINCT** on the same column. It depends on `null` values, as we've seen. If there is maximum one `null` value in the column, they are the same, otherwise they are not.
{: .prompt-warning}

![Nulls are same](/assets/img/posts/2022-08-30-sql-unique-vs-distinct/nulls-are-same-are-not.jpg){: .shadow style="max-width: 280px; width: 90%" }
