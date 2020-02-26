# Admin training
## Aircloak Insights

Telefonica – March 2nd 2020

---

# Agenda

1. What is Aircloak and what are its parts
1. Anonymization primer and data source configuration
1. Supporting the users and yourselves 
1. System configuration and maintenance

---

## Session 1:
# What is Aircloak?

--

## Aircloak 

- is an SQL query proxy <!-- .element: class="fragment"-->
- performs dynamic anonymization at runtime <!-- .element: class="fragment"-->
- no risk of accidental data disclosure <!-- .element: class="fragment"-->

--

![Image](content/images-admin/graphic-overview-new.svg) <!-- .element: style="max-height:600px;border:none;" -->

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
  - Explicitly given informed concent
  - Not working with data
  - Breaking the law
  - Anonymizing data

--

## Anonymization is difficult

- Anonymization tends to be done on a case-by-case basis
- The turnaround time from idea to data tends to be in weeks or months rather than minutes
- Data Protection Officers involuntarily become "bad wolfs" rather than enablers

--

## Aircloak

- Data agnostic anonymization
- Use case agnostic anonymization
- Install once, run any query
- Real-time access to work on live production data
- No need to maintain a separate copy of data

--

## Terms and how they differ

- Pseudonymization
- Anonymization

--

### Raw data

| UserID | Age | Salary | Name    |
|--------|----:|-------:|---------|
| 1      | 10  |     0  | Alice   |
| 2      | 20  | 20000  | Bob     |
| 3      | 30  | 50000  | Cynthia |

<!-- -- data-transition="none" data-background-transition="none" -->

--

<!-- -- data-transition="none" data-background-transition="none" -->

### Pseudonymized data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| A      | 10  |     0  | Alvar   |
| B      | 20  | 20000  | Borg    |
| C      | 30  | 50000  | Crantor |

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

## Components

--

## Insights Air

- Authentication (via LDAP/AD)
- Authorization (based on groups from LDAP/AD)
- Audit logging
- Load balancing between Insight Cloak instances

--

## Insights Cloak

- Core anonymization engine
- Interacts with the database containing the raw data

--

## Data source

- Roughly corresponds to a database in a database server
- Configured at the level of Insights Cloak
- Specifies which tables should be exposed
- Specifies anonymization related parameters

--

![Image](content/images-admin/single-low.png) <!-- .element: style="max-height:600px;border:none;" -->

---

<!-- -- data-transition="slide-in none-out" -->

## Scaling

![Image](content/images-admin/single-low.png) <!-- .element: style="max-height:600px;border:none;" -->

--

<!-- -- data-transition="none" -->

## Scaling

![Image](content/images-admin/multi-low.png) <!-- .element: style="max-height:600px;border:none;" -->

--

<!-- -- data-transition="none" -->

## Scaling

![Image](content/images-admin/single-high.png) <!-- .element: style="max-height:600px;border:none;" -->

--

<!-- -- data-transition="none" -->

## Scaling

![Image](content/images-admin/multi-high.png) <!-- .element: style="max-height:600px;border:none;" -->

---

## Performance impact

- Queries are slower than when using Toad
- Aircloak's query rewriting increase the query complexity

--

## Query rewrite example

![Image](content/images-admin/rewrite.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Where is the time spent

~95% of query execution time spent in Oracle

--

## Improving performance

- No scale-out parallelism at the level of the Insights Cloak
  - Multiple instances to avoid single point of failure
- Caching intermediate results with CTAS 
- Reducing dataset (for example `SYSTEM_STACK_ID = X`) for explorative queries

---

## Setup at Telefonica

--

## Infrastructure

- Kubernetes
- Separate Postgres instance
- LDAP

--

## Maintenance

- All configuration files are maintained on GitLab
- Insights Air and Insights Cloak have separate configuration files 
- Data sources have individual configuration files

Details on the configuration files later today.

--

## Upgrading and deploying - today

1. Receive `air` and `cloak` container images from Aircloak
2. Push them to the Telefonica Docker registry
3. Alter Kubernetes Pod definitions
4. Redeploy services

--

## Upgrading and deploying - future

1. Telefonica's docker registry mirrors that of Aircloak
3. Alter Kubernetes Pod definitions to deploy new version
4. Redeploy services

---








## Session 2:
# Anonymization primer and data source configuration

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

## User ids and anonymization

- Aircloak builds groups to allow individuals to hide
- Aircloak uses __user ids__ to determine if a group is large enough
- The data source configuration specifies which column contains the __user ids__

---

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

## Check-list: personal or not

- Does the data contain PII? ---> personal
- Does the data belong to an individual? ---> personal
- Is the data general facts? ---> non-personal

--

## Personal or not quiz

--

## Personal or not - 1

A table containing `names` and `phone numbers`.

Personal <!-- .element: class="fragment" -->

--

## Personal or not - 2

A table with a mapping from `country name` to `national anthem`.

Non-personal <!-- .element: class="fragment" -->

--

## Personal or not - 3

A table with a mapping from a `customer` to `national anthem`.

Personal <!-- .element: class="fragment" -->

--

## Personal or not - 4

A table containing a `contract_id` for a cellphone contract.

Personal <!-- .element: class="fragment" -->

--

## Personal or not - 5

A table containing a `product_id` and `product_description`.

Non-personal <!-- .element: class="fragment" -->

--

## Personal or not - 6

A table containing a mapping from `contract_id` to `product_id`

Personal <!-- .element: class="fragment" -->

--

## Queries over personal data must include a user-id column

But what is the table does not have a __user id__ column?

---

## Keys

- Keys specify which column contains the __user_id__.

<!-- -- data-transition="slide-in none-out" -->

--

## Keys

- Keys specify which column contains the __user_id__.

Additionally:

- Specify how personal tables can be joined
- Specify how non-personal and personal tables can be joined
- Usually (but not restricted to) foreign keys 

<!-- -- data-transition="none-in slide-out" -->

--

![Image](content/images/data-model-3.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Keys - user-id

- Key name: `user_id`
- Reserved for the columns that contain the logical user identifier

---

## Configuring data sources

--

## Data source definition

- Data sources are defined in json files
- One data source per file

-- 

## Data source skeleton

```json
{
  "name": "DataSourceName",
  "driver": "oracle",
  "parameters": {
    "hostname": string,
    "port": integer,
    "username": string,
    "database": string,
    "password": string
  },
  "analyst_tables_enabled": true,
  "tables": {
    ...
  }
}
```

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

## A restricted query:

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

## ... within an unrestricted query!

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

# Q&A