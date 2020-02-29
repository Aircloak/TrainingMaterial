# Analyst training
## Aircloak Insights

Telefonica – 3rd of March 2020

---

# Agenda

1. Anonymization
1. Querying with Aircloak
1. Practical session
1. Telefonica specific details

--

## Sessions

- Start on the hour
- Each session takes ~45 minutes
- 15 minute break between sessions

---

## Introductions

--

## We are

- Felix 
- Sebastian 

--

## You are

Quick round. What is your:
- name
- role

---

Before we start

## What is Aircloak?

--

## Aircloak 

- is an SQL query proxy <!-- .element: class="fragment"-->
- performs dynamic anonymization at runtime <!-- .element: class="fragment"-->
- no risk of accidental data disclosure <!-- .element: class="fragment"-->

--

## It is accessible through ...

--

<!-- -- data-transition="none" data-background="content/images/aircloak-interface.png" -->

## a browser interface <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


--

<!-- -- data-transition="none" data-background="content/images/rest-api.png" -->

## a REST API <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


--

<!-- -- data-transition="none" data-background="content/images/psql.png" -->

## Postgres query interface <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->

---

## Why should we care?

--

## What value does it bring?

Every item that complicates a system should justify its existence

--

## Gaining access to data

- Data protection is important – yet difficult
- Alternatives are:
  - Legitimate interest
  - Performance of contract
  - Explicitly given informed consent
  - Not working with data
  - Breaking the law
  - Anonymizing data

--

## Anonymization

Anonymization allows you to not have to argue about legitimate interest or consent.

--

## Anonymization is difficult

- Anonymization tends to be done on a case-by-case basis
- The turnaround time from idea to can be in weeks or months rather than minutes
- Analyst spend time manually anonymizing data rather than do analysis

--

## Aircloak

- Data agnostic anonymization
- Use case agnostic anonymization
- Install once, run any query
- Real-time access to work on live production data
- At the cost of slightly harder query writing experience

---

## Session 1: 
# Anonymization

---

## Terms and how they differ

- Pseudonymization
- Anonymization

--

### Raw data

| UserID | Age | Salary | Name    |
|--------|----:|-------:|---------|
| 1      | 11  |    10  | Alice   |
| 2      | 24  | 21000  | Bob     |
| 3      | 39  | 50300  | Cynthia |

<!-- -- data-transition="none" data-background-transition="none" -->

--

<!-- -- data-transition="none" data-background-transition="none" -->

### Pseudonymized data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| A      | 11  |    10  | Alvar   |
| B      | 24  | 21000  | Borg    |
| C      | 39  | 50300  | Crantor |

--

### Anonymization

In this particular example: 

> There were roughly `3` users in the dataset

--

### Anonymization

More generally some form of aggregation and distortion:

| Age | Salary | Name | Noisy count | 
|----:|-------:|------|------------:|
| 10  |     0  | *    |         250 |
| 12  |     0  | *    |         201 |
| 13  |  1-200 | *    |          43 |
| ... |    ... | *    |         ... |
| 84  | 200000 | *    |          14 |

--

### Notice:

- Pseudonymous yields per user data
- Pseudonymous data is considered personal
- Information about individuals can often be inferred

--

### Confusing these categories can lead to disaster

---

## Anonymization and Aircloak

---

## Low count filtering

--

## Low count filtering

If some descriptors are common to multiple users (~5) then it is generally:
- not identifying
- safe to share

--

## Scenario:

You walk into a meeting room and find:
- a mess of half-eaten food and spilled coffee
- a notebook with the owners name written on it

--

## Consider the following prior information:

- a) 1 person was in the room
- b) 25 people were in the room

--

## 1 person

- You _do know_ who the owner of the notebook is
- You know with _high certainty_ that this person also made the mess

--

## 25 people

- You _do know_ who the owner of the notebook is
- You _do not know_ who made the mess

<!-- -- data-transition="slide-in none-out" -->

--

<!-- -- data-transition="none-in slide-out" -->

## 25 people

- You _do know_ who the owner of the notebook is
- You _do not know_ who made the mess

You can hide in a crowd!

--

## Quiz

--

## Quiz
### Scenario: We know of all purchases of an "Mega clean 2000" (an obscure cleaning product)

--

### Quiz 1 
#### Is "Mega clean 2000" ok to reveal? 

| UserID | Product         |
|-------:|-----------------|
|      1 | Mega clean 2000 |
|      1 | Mega clean 2000 |
|      1 | Mega clean 2000 |
|      1 | Mega clean 2000 |
|      1 | Mega clean 2000 |
|      1 | Mega clean 2000 |

<span class="fragment">
No. Only a single person bought the product. There is no crowd to hide in.
</span>

--

### Quiz 2
#### Is "Mega clean 2000" ok to reveal? 

| UserID | Product         |
|-------:|-----------------|
|      1 | Mega clean 2000 |
|      2 | Mega clean 2000 |
|      3 | Mega clean 2000 |
|      4 | Mega clean 2000 |
|      5 | Mega clean 2000 |
|      6 | Mega clean 2000 |

<span class="fragment">
Yes. ~5 users bought the product. They are all hiding in a crowd.
</span>

---

## Fine grained values

If you have fine grained data (like for example a `timestamp`) then every value becomes a "Mega clean 2000"-equivalent.

--

### Example: fine grained values

| UserID | BirthTime              |
|--------|-----------------------:|
|      1 | 1980-01-01 12:15:01.02 |
|      2 | 1980-02-01 02:21:00.12 |
|      3 | 1980-02-01 02:50:05.99 |
| ...    | ...                    |
|      N | 1980-12-30 18:30:05.19 |

--

![Image](content/images/lcf-g1.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

![Image](content/images/lcf-g2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/lcf-g3.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/lcf-g4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/lcf-g5.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/lcf-g6.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## What happens if we add a second category of values?

- `(BirthTime, Gender)` 
- It is more specific
- The resulting groups are smaller

--

![Image](content/images/lcf-g7.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Tradeoff 
### More columns, less results

--

## Tradeoff 
### More columns, less results

`(zip-code, date of birth, gender)` uniquely identifies 67% of all US citizens (of which there are 329 million)!

--

## There is a hidden attribute in your queries!

--

## You think of your data as

| Gender | BirthTime              |
|--------|-----------------------:|
| M      | 1980-01-01 12:15:01.02 |
| M      | 1980-02-01 02:21:00.12 |
| F      | 1980-02-01 02:50:05.99 |
| ...    | ...                    |
| F      | 1980-12-30 18:30:05.19 |

<!-- -- data-transition="slide-in none-out" -->

--

## But in reality it is

| Is our customer | Gender | BirthTime              |
|:----------------|--------|-----------------------:|
| True            | M      | 1980-01-01 12:15:01.02 | 
| True            | M      | 1980-02-01 02:21:00.12 |
| True            | F      | 1980-02-01 02:50:05.99 |
| True            | ...    | ...                    |
| True            | F      | 1980-12-30 18:30:05.19 |

<!-- -- data-transition="none-in slide-out" -->

--

# In summary

The more columns your query includes, the higher the chance
- that the combination is identifying
- that Aircloak will filter the data out as part of anonymization

---

## Interlude - Terminology

--

## Noise

--

<!-- -- data-transition="none" data-background="content/images/ds-without-noise.jpg" -->

## Aspiring data scientists <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->

--

<!-- -- data-transition="none" data-background="content/images/ds-with-noise.jpg" -->

## Maybe aspiring data scientists <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->

--

## Noise for numbers

If we say we add a noise of 10 to the value 100, we mean: 

> I will pick a random number between -10 and +10 and add it to 100.

--

## Noise for numbers 
### Random number

The random number could be:

- -10
- 4
- 9

<!-- -- data-transition="slide-in none-out" -->

--

## Noise for numbers 
### Random number

The random number could be:

- -10 ---> 100 + (-10) = 90
- 4 ---> 100 + 4 = 104
- 9 ---> 100 + 9 = 109

<!-- -- data-transition="none-in slide-out" -->

--

## How to choose a random number

The random number must be taken from some distribution

--

## Uniform distribution

Any value between -10 and +10 are equally likely

--

## Gaussian/Normal distribution

Values closer to 0 are more likely.

--

## Gaussian/Normal distribution

![Image](content/images/gaussian-dist.png) <!-- .element: style="max-height:600px;border:none;" -->

---

# Aggregate statistics

--

### A histogram of salaries

![Image](content/images/salaries-1.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

### How could an aggregate be dangerous?

--

### Aggregates leak information

```sql
SELECT count(distinct userId)
FROM users
```

vs

```sql
SELECT count(distinct userId)
FROM users
WHERE streetAddress <> 'Thorst. 18, apt. 5'
```

The difference reveals whether or not the inhabitant of the appartment is in our dataset or not.

--

### Aggregates leak information

```sql
SELECT sum(salary)
FROM users
```

vs

```sql
SELECT sum(salary)
FROM users
WHERE position <> 'CEO'
```

The difference yields the salary of the CEO.

--

### Low count filter would suppress salaries

![Image](content/images/salaries-2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Aggregates are treated differently

![Image](content/images/salaries-3.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### How about extreme values?

![Image](content/images/salaries-4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### How about extreme values?

![Image](content/images/salaries-5.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Extreme values are shifted

![Image](content/images/salaries-6.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Extreme values are shifted

![Image](content/images/salaries-7.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

### Adding noise

![Image](content/images/aggregate-noise-1.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

### Noise proportional to "heavy" users

![Image](content/images/aggregate-noise-2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Add noise

![Image](content/images/aggregate-noise-3.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Where the answer might be

![Image](content/images/aggregate-noise-4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Quantifying the noise

`_noise` companion functions

--

## Quantifying the noise

| aggregate | noise function |
|-----------|----------------|
| count     | count_noise    |
| sum       | sum_noise      |
| avg       | avg_noise      |
| stddev    | stddev_noise   |
| min       |                |
| max       |                |

<!-- -- data-transition="slide-in none-out" -->

--

## Quantifying the noise

| aggregate | noise function |
|-----------|----------------|
| count     | count_noise    |
| sum       | sum_noise      |
| avg       | avg_noise      |
| stddev    | stddev_noise   |
| min       | ???            |
| max       | ???            |

<!-- -- data-transition="none-in slide-out" -->

---

## User ids and anonymization

- Aircloak builds groups to allow individuals to hide
- Aircloak uses user ids to determine if a group is large enough

--

## User ids at Telefonica

- What is the user id depends on the data source
- Columns such as `client_id`, `party_id`, `contract_id`, and `vendor_id`

--

## Types of tables

![Image](content/images/personal-vs-non-personal.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/data-source.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

![Image](content/images/data-model-1.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/data-model-2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/data-model-3.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Types of tables

- **personal**: require anonymization
- **non-personal**: usually do not require anonymization

--

## Queries over personal data must include a user-id column

--

## Example 1

```sql
SELECT users.userId, users.name, count(*)
FROM users
GROUP BY users.userId, users.name
```

--

## Example 2

```sql
SELECT users.name, count(*)
FROM users
GROUP BY users.name
```

--

## Example 3

```sql
SELECT count(purchases.id)
FROM purchases
```

--

## Example 4 - not valid

```sql
SELECT count(lineItems.id)
FROM lineItems
```

Remember: each query over a personal table must have a user-id column.

--

## Example 4 - solution

Include a `JOIN` to a table with a `UserId` column:

```sql
SELECT count(*)
FROM lineItems inner join purchases
  ON lineItems.purchaseId = purchases.purchaseId
```

--

## non-personal tables

```sql
SELECT products.productName
FROM products
```

No personal data = no anonymization required

--

## Unless...

non-personal tables become personal if combined with a personal table!

![Image](content/images/tainted.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Combining table types

Not allowed:

```sql
SELECT products.productName, count(*)
FROM products INNER JOIN lineItems
  ON products.productId = lineItems.productId
```

--

## Combining table types

Allowed:

```sql
SELECT products.productName, count(*)
FROM products INNER JOIN lineItems
  ON products.productId = lineItems.productId
  INNER JOIN purchases 
  ON lineItems.purchaseId = purchases.purchaseId 
```

---

## Types of queries

--

![Image](content/images/query-types-1.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

![Image](content/images/query-types-2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/query-types-3.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

![Image](content/images/query-types-4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Why do I need to know

- Knowing when anonymization takes place is important for good query results
- Unrestricted queries do not underly Aircloak query restrictions

--

## Anonymizing vs non-anonymizing

--

### Non-anonymizing restricted query

![Image](content/images/query-transformation-1.png) <!-- .element: style="max-height:600px;border:none;" -->

--

### Non-anonymizing restricted query

```sql
SELECT 
  purchases.userId, 
  count(distinct purchases.PurchaseId) as numPurchases
FROM purchases
GROUP BY purchases.userId
```

--

### Anonymizing restricted query

![Image](content/images/query-transformation-2.png) <!-- .element: style="max-height:600px;border:none;" -->

--

### Anonymizing restricted query

```sql
SELECT count(distinct purchases.PurchaseId) as numOfPurchases
FROM purchases
```

--

## Examples of query types

--

### Restricted query
## Anonymizing 

- A query that **aggregates or groups** across multiple distinct users

```sql
SELECT users.name, count(users.userId) as numUsers
FROM users
GROUP BY users.name
```

--

### Restricted query
## Non-anonymizing 

- Only useful as sub-queries!
- Aircloak will try to anonymize the results if it is the top-level query

--

### Restricted query
## Non-anonymizing 

- Operates on the data of a single user

```sql
SELECT purchases.UserId, count(*) as numPurchases
FROM purchases
GROUP BY purchases.UserId
```

--

## Unrestricted queries

- A query that operates on non-personal data 
- A query that operates on an already anonymous result

```sql
SELECT products.productName
FROM products 
```

--

## Let's combine them!

--

## A restricted query

```sql
-- Non-anonymizing restricted query
SELECT purchases.userId, lineItems.productId, count(*) as purchasedByUser
FROM lineItems li INNER JOIN purchases p
  ON li.purchaseId = p.purchaseId
GROUP BY purchases.userId, lineItems.productId
```

--

## ... within an anonymizing query

```sql
-- Anonymizing restricted query
SELECT 
  perUser.productId, 
  perUser.numPurchases, 
  count(*)
FROM (
  -- Non-anonymizing restricted query
  SELECT purchases.userId, lineItems.productId, count(*) as numPurchases
  FROM lineItems li INNER JOIN purchases p
    ON li.purchaseId = p.purchaseId
  GROUP BY purchases.userId, lineItems.productId
) as perUser
GROUP BY
  perUser.productId, 
  perUser.numPurchases
```

--

![Image](content/images/anon-graphed.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## An anonymizing query

```sql
-- Anonymizing query
SELECT lineItems.productId, count(*) as totalNumPurchases
FROM lineItems li INNER JOIN purchases p
  ON li.purchaseId = p.purchaseId
GROUP BY lineItems.productId
```

--

## ... within an unrestricted query

```sql
-- Unrestricted query
SELECT productId
FROM (
  -- Anonymizing query
  SELECT lineItems.productId, count(*) as totalNumPurchases
  FROM lineItems li INNER JOIN purchases p
    ON li.purchaseId = p.purchaseId
  GROUP BY lineItems.productId
) productPopularity
WHERE totalNumPurchases > 100
```

--

## Combining all three query types

```sql
-- Unrestricted query
SELECT anonResult.productId
FROM (
  -- Anonymizing restricted query
  SELECT 
    perUser.productId, 
    perUser.numPurchases, 
    count(*)
  FROM (
    -- Non-anonymizing restricted query
    SELECT purchases.userId, lineItems.productId, count(*) as numPurchases
    FROM lineItems li INNER JOIN purchases p
      ON li.purchaseId = p.purchaseId
    GROUP BY purchases.userId, lineItems.productId
  ) as perUser
  GROUP BY
    perUser.productId, 
    perUser.numPurchases
) anonResult
HAVING SUM(anonResult.numPurchases) * SUM(anonResult.count) > 100
```

--

![Image](content/images/query-types-transition.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Quiz time

--

## Data model refresher

![Image](content/images/data-model-3.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Restricted or un-restricted?

```sql
SELECT count(*)
FROM products
```

--

## Anonymizing or non-anonymizing 

```sql
SELECT count(purchases.UserId)
FROM purchases
```

--

## Anonymizing or non-anonymizing sub-query

```sql
SELECT purchases.userId, count(lineItems.purchaseId) as numLineItems
FROM purchases p INNER JOIN lineItems li
  ON p.purchaseId = li.purchaseId
GROUP BY purchases.userId
```

--

## Good job!

---

## Break








---

## Session 2: 
# Querying with Aircloak

---

## SQL

Aircloak supports a subset of standard SQL.

Docs are a useful resource.

---

## Differences to Oracle 

- Limited OR functionality
- No WINDOW function support
- Some function names differ slightly
- JOINs

---

## Differences to Oracle - truncation of timestamps

- `date_trunc` for truncating for dates and times 

| Oracle                  | Aircloak                 |
|-------------------------|--------------------------|
| trunc(c, 'yyyy')        | date_trunc('year', c)    |
| trunc(c, 'q')           | date_trunc('quarter', c) |
| trunc(c, 'mm')          | date_trunc('month', c)   |
| trunc(c, 'dd')          | date_trunc('day', c)     |
| trunc(c, 'hh')          | date_trunc('hour', c)    |
| trunc(c, 'mi')          | date_trunc('minute', c)  |
| cast(c as timestamp(0)) | date_trunc('second', c)  |

--

## Importance of `date_trunc`

![Image](content/images/lcf-g4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

## Importance of `date_trunc`

![Image](content/images/lcf-g5.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Differences to Oracle - `bucket`

- Aircloak has a `bucket` function
- It groups values to a grid
- It used like this: `bucket(column by gridSize)`

--

## Visualizing `bucket`

![Image](content/images/bucket-before.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

## Visualizing `bucket`

![Image](content/images/bucket-down.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

## Visualizing `bucket`

![Image](content/images/bucket-mid.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

## Visualizing `bucket`

![Image](content/images/bucket-up.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Importance of `bucket`

![Image](content/images/lcf-g4.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="slide-in none-out" -->

--

## Importance of `bucket`

![Image](content/images/lcf-g5.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Differences to Oracle 

| Oracle      | Aircloak |
|-------------|----------|
| STDDEV_SAMP | stddev   |
| VAR_SAMP    | variance |

--

## Differences to Oracle - `ilike`

- Aircloak supports case insensitive LIKE

| Oracle                    | Aircloak            |
|---------------------------|---------------------|
|       col LIKE 'pattern'  | col  LIKE 'pattern' |
| LOWER(col) LIKE 'pattern' | col ILIKE 'pattern' |

--

## JOINs 

```SQL
-- ORACLE join syntax, esp. for outer joins
    from my_first_table a
       , my_second_table b
   where a.my_dt(+) >= to_Date('01082019','DDMMYYYY')
     and a.my_dt(+) < to_Date('01092019','DDMMYYYY')
     and b.main_id = 97
     and b.other_id = 1
     and a.second_pk(+) = b.second_pk
-- becomes
      from my_first_table a
right join my_second_table b
        on b.second_pk = a.second_pk
       and a.my_dt      >= DATE('2019-08-01')
       and a.my_dt       < DATE('2019-09-01')
     where b.main_id = 97
       and b.other_id = 1
```

---

## Common pitfalls - *

```sql
SELECT * FROM table
```

<span class="fragment">
Remember: Aircloak suppresses column combinations that aren't shared by sufficiently many users. With `SELECT *` all rows end up identifying 
</span>
<!-- -- data-transition="slide-in none-out" -->

--

## Common pitfalls - *

```sql
SELECT * FROM table
```

![Image](content/images/selectstar.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none-in slide-out" -->

--

## Common pitfalls - LIMIT X

```sql
SELECT columnA, columnB, columnC
FROM table
LIMIT 10
```

<span class="fragment">
Aircloak can only determine the rows to return after the full anonymization has taken place.
</span>

---

## Accounting for effects of anonymization 

- Identifying information is supressed
- Aircloak cannot tell you what this data was
- We give you an anonymizing estimate of how much data was suppressed

--

## Accounting for effects of anonymization - `*`

![Image](content/images/star-row.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Aggregates over small amounts of data

| User Id | Item | Mileage  |
|--------:|------|---------:|
| 1       | Car  | -10      |
| 2       | Car  | 80       |
| 3       | Car  | 999999   |

--

## Aggregates over small amounts of data

```sql
SELECT item, aggregate(mileage)
FROM items
GROUP BY item
```

- Enough data to reveal that "Car" exists in the dataset
- Not enough data to produce a meaningful aggregate of mileage

--

### What Aircloak reports

| aggregate       | return value |
|-----------------|-------------:|
| count(mileage)  | 2            |
| avg(mileage)    | null         |
| sum(mileage)    | null         |
| stddev(mileage) | null         |
| max(mileage)    | null         |
| min(mileage)    | null         |

> This property exists in my dataset. But I CANNOT make any statistical assesment of any aggregate properties

--

### count = 2?

The SQL standard doesn't allow `null` as a return value for `count`. We would never report something that was not at the _very least_ shared by two or more users.

Read a count of 2 as meaning: 

> This property exists in my dataset. I DO NOT know for how many users.

--

### What about X_noise?

| aggregate       | return value | _noise |
|-----------------|-------------:|-------:|
| count(mileage)  | 2            | null   |
| avg(mileage)    | null         | null   |
| sum(mileage)    | null         | null   |
| stddev(mileage) | null         | null   |
| max(mileage)    | null         |        |
| min(mileage)    | null         |        |

--

## Filtering insignificant aggregates - low counts

You can filter out aggregates that are not meaningful.

```sql
SELECT salary, count(*)
FROM salaries
GROUP BY salary
HAVING count(*) > 10
```

--

## Filtering insignificant aggregates - noisy results

You can filter out aggregates where the noise dominates.

```sql
SELECT salary, count(*)
FROM salaries
GROUP BY salary
-- Signal to noise ratio better than 10
HAVING count(*) / count_noise(*) > 10
```

---

## Working with freeform text

User provided text could be considered as a relatively strong pseudorandom numbger generator...

```sql
SELECT freetextColumn, ...
FROM table
```

All results will get anonymized away

--

## Working with freeform text

![Image](content/images/lcf-g4.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Extract parts of text

- `substring`
- `left`
- `right`

--

## Filter based on text

- `like`
- `ilike`

```sql
SELECT some, columns, aggregates(...)
FROM tables...
WHERE freetextColumn ILIKE '%thank you%'
GROUP BY ...
```

--

## Building co-horts

```sql
SELECT DISTINCT userId 
FROM table
WHERE freetextColumn ILIKE '%thank you%'
```

--

## Building co-horts

```sql
SELECT some, columns, aggregates(...)
FROM table INNER JOIN (
  SELECT DISTINCT userId 
  FROM table
  WHERE freetextColumn ILIKE '%thank you%'
) thankfulUsers ON table.userId = thankfulUsers.userId
GROUP BY ...
```

--

## Extract common logic 

- into views
- into table ("create table as select")

--

## Extract common logic 

# Demo

---

## Restrictions

- All restrictions for anonymization purposes
- Over time some might be removed or softened.
- Restrictions only apply in anonymizing and restricted queries.

--

## Most commonly noticed restrictions

- OR-functionality
- Ranges

--

## OR-functionality

- heavily restricted
- support for `IN` in the `WHERE`-clause
- support for simplified `CASE`-statements
  - inside aggregate in the anonymizing query
  - as selected column in the anonymizing query

--

## Ranges

- Ranges containing constants must be bounded

```sql
-- Allowed - bounded
WHERE age BETWEEN 10 and 20

-- Not allowed - not bounded
WHERE age > 10
```

--

## Ranges

- Column inequalities are allowed

```sql
-- Allowed - no constant 
WHERE salary < age
```

--

## Ranges - snapping

- Range boundaries must fall on a grid "1, 2, 5"-grid

--

## Ranges - snapping - 1, 2, 5

Allowed values:
- ..., 0.1, 0.2, 0.5, 1, 2, 5, 10, 20, 50, 100, 200, 500, ...

--

## Ranges - snapping - 1, 2, 5

```sql
WHERE age BETWEEN 11 and 19
```

becomes

```sql
WHERE age BETWEEN 10 and 20
```

--

## Ranges - snapping - 1, 2, 5

Similarly dates are also snapped to a 1, 2, 5 grid.

--

## Full list of restrictions are in the documentation

---

## What is next?

After the break we will have a practical session.

First to solve the task wins a Grand Price!

---

# Break

---

## Session 3: 
# Practical session!

---

## Scenario

- We are data scientists at Berka Bank 
- We want to figure out:
  - Which customer group brings in the most revenue as a group
  - Which customer group brings in the most revenue per member
- "Customer group" is defined by their credit card type

--

### "Facts"

- We charge 1% interest on credit card transactions
- The transactions we care about are the "expense"-transactions (`type = 'VYDAJ'`)

--

## Data model

![Image](content/images/task3-schema.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Sample Jupyter Notebook

https://download.aircloak.com/analyst-training/

--

## Where is the data?

- URL: https://demo.aircloak.com
- Username: your email address
- Password: `telefonica1234`

--

## Data model + task

<div>
  <img src="content/images/task3-schema.png"
    style="border: none; max-width: 22vw; float:left"
  >

  <div style="float:left; width: 22vw; padding: 2rem">
    <ul>
      <li>We charge 1% interest on credit card transactions (`type = 'VYDAJ'`)</li>
      <li>Which customer group (credit card type) brings in the most revenue: as a group, per customer</li>
    </ul>
  </div>
</div>

--

## Solution walk-through 

---

# Break

---

## Session 4: 
# Practical details

---

## Telefonica's Aircloak instance

- Web interface: http://demucdnhkm01.dcn.de.pri.o2.com:30080
- Postgres interface: demucdnhkm01.dcn.de.pri.o2.com:300801
  - See sample Python notebook from practical session
  - https://download.aircloak.com/analyst-training/
- Documentation: http://demucdnhkm01.dcn.de.pri.o2.com:30080/docs

---

## How do I get access

1. Open a JIRA request in DWH Services Infrastructure (DWHI) stating the names of the data sources
2. "Grant access to aircloak"
3. Add component "aircloak"

---

## I need other data, what to do

1. Open a JIRA request in DWH Services Infrastructure (DWHI) stating the DBMS object names
2. "Include objects into Aircloak",
3. Add component "aircloak"

---

## I need a specific feature

There are three ways working around missing features

1. __Fast__: Sometimes it's a matter of reformulating the query
2. __Medium__: Create views in Oracle and expose the views through Aircloak
3. __Slow/Uncertain__: Request the addition of the feature

---

## Oracle DB Link / DB Gateway

- not yet final, one DB-Link exists but we still have to work out how to do it properly

---

## Internal resources

- Internal handbook with learnings

> K:\Business Intelligence Center\02_DATAWAREHOUSING_Team\40 Projects\Anonymisierung\05_Aircloak\13_Handbook

---

## Getting help

- Telefonica offers 1st level support (Jira)
  - DWHI, add component "aircloak"
- Problems can be escalated to Aircloak (Jira)
  - please refer to your 1st level support ~(Jira)~ imho Aircloak's JIRA should only be operated by 
    our 1st level support when you have problems, you can potentially provide a debug report to you 
    DWH SEs(see next slide)

--

### Debug exports

Often debug exports are necessary to troubleshoot specific problems.
A debug export contains:

- your query
- a snapshot of the schema
- a log of what happened in the anonymization engine
- the query generated by Aircloak for Oracle

--

### Enabling debug exports

![Image](content/images/de-menu.png) <!-- .element: style="max-height:600px;border:none;" -->

<!-- -- data-transition="slide-in none-out" --> 

--

### Enabling debug exports

![Image](content/images/de-settings-pre.png) <!-- .element: style="max-height:600px;border:none;" -->

<!-- -- data-transition="none" --> 

--

### Enabling debug exports

![Image](content/images/de-settings-post.png) <!-- .element: style="max-height:600px;border:none;" -->

<!-- -- data-transition="none" --> 

--

### Enabling debug exports

![Image](content/images/de-button.png) <!-- .element: style="max-height:600px;border:none;" -->

<!-- -- data-transition="none-in slide-out" --> 

---

## Further reading

- Understanding query results (in docs)
- Best practises (in docs)
- https://training.aircloak.net - shows anonymized and non-anonymized results side by side

---

# Q&A