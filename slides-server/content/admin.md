# Admin training
## Aircloak Insights

Telefonica – 2nd of March 2020

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
- The turnaround time from idea to anonymized data can be measured in weeks or months rather than minutes
- Data scientists spend time anonymizing data rather than doing analysis
- Data Protection Officer becomes "bad wolf" rather than enabler

--

## Aircloak

- Data and use case agnostic anonymization
- Install once, run any query
- Real-time access to production data
- No need to maintain a separate copy of data
- At the cost of slightly harder query writing experience

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

> There were roughly `3` users in the data set

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
- Reducing data set (for example `SYSTEM_STACK_ID = X`) for exploratory queries

---

## Setup at Telefonica

--

## Infrastructure

- Kubernetes
- Postgres database instance for Insights Air
- LDAP

--

## Maintenance

- All configuration files (including data source definitions) are maintained on GitLab
- Insights Air and Insights Cloak have separate configuration files 
- Data sources have individual configuration files

Details on the configuration files later today.

--

## Starting / stopping Aircloak

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

--

## CAUTION!

- Insights Air automatically migrates the database schema upon being run!
- Downgrading to a previous version is non-trivial
- __Take a database backup before upgrading!__

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

## How to choose a user-id

--

## Choosing a user id

Ideally it is obvious. That is when there is a `user-id`, `client-id`, or `customer-id` column.

Reality is seldom obvious.

--

#### Choosing a user id 
## Hierarchies

- Client
- Party
- Contract

Assumption: there is a one to many relationship between the hierarchies. One client has many parties, one party has many contracts, etc.

--

#### Choosing a user id 
## Hierarchies gotchas

You cannot safely expose tables with data at the level of the higher up hierarchies. These will not be adequately protected. 

Example: If you use __Party__ as the source of your user id, then tables at the level of __Client__ are not protected.

--

#### Choosing a user id 
## Hierarchies tradeoff

Choosing the top-level hierarchy is generally ideal. In practice this hierarchy might be sparsely populated. Aircloak filters out data where the user-id is empty.

It's a tradeoff between which tables can be made available and how much of the data in the exposed tables is available for analysis.

--

#### Choosing a user id 
## Multiple entity classes

The data set might contain different classes of entities that should be protected. For example vendors vs clients.
Aircloak is designed to protect one type of entity.

__OK IFF__: the tables do not overlap or contain data about more than one entity class

--

#### Choosing a user id 
## Multiple entity classes - solution

Create separate data sources per entity class. Ensure the data exposed is only referring to the specified entity class and not another.

--

#### Choosing a user id 
## Multiple users per row

Aircloak is built on the assumption that a database row belongs to a single individual.

This does not hold true for transaction data:
- A called B
- A sent money to B, etc

--

#### Choosing a user id 
## Multiple users per row - opt. A

The solution today is to split such tables in two such that each half only contains the data belonging to one of the users. Shared data can be made available in all tables.

--

#### Choosing a user id 
## Multiple users per row - opt. B

The solution in the future is to split such tables in multiple parts. 
Each part contains the data for one of the user, and has the columns containing data about the other users grey-listed.

This will be possible in Aircloak version `20.2`

---

## Configuring data sources

--

## Data source definition

- Data sources are defined in json files
- One data source per file

--

## JSON

- JSON is a structured way to encode data
- It consists of "objects" which are a set of key-value pairs
- The values can themselves be objects in their own right

--

## JSON - Example

An object is denoted by curly brackets `{` and `}`:

```json
{

}
```

<!-- -- data-transition="slide-in none-out" -->

--

## JSON - Example

Objects contain key-value pairs:

```json
{
  "organization_name": "Telefonica"
}
```

<!-- -- data-transition="none" -->

--

## JSON - Example

Objects contain key-value pairs:

```json
{
  "organization_name": "Telefonica"
  "office": "Munich",
  "metadata": {
    "uses": "Aircloak",
    "running": true,
    "instances": 1
  }
}
```

<!-- -- data-transition="none" -->

--

## JSON - Example

Values can also be lists:

```json
{
  "organization_name": "Telefonica",
  "office": "Munich",
  "metadata": {
    "uses": "Aircloak",
    "running": true,
    "instances": 1
  },
  "brands": ["Telefonica", "O2", "Others..."]
}
```

<!-- -- data-transition="none-in slide-out" -->

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
    ... the interesting part ...
  }
}
```

--

## Tables

```json
"tables": {
  "tablenameShownInAircloak": {
    "db_name": "tablenameUsedInDatabase",
    "content_type": "personal" | "non-personal",
    ...
  },
  ...
}
```

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

non-personal tables become personal if combined with a personal table!

![Image](content/images/tainted.png) <!-- .element: style="max-height:600px;border:none;" -->

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

- How do I configure which is the __user id__ column?
- What if the table does not have a __user id__ column?

--

## Tables

```json
"tables": {
  "tablenameShownInAircloak": {
    "db_name": "tablenameUsedInDatabase",
    "content_type": "personal" | "non-personal",
    "keys": [{"key_type_1": "column_name_1"}, ...],
    ...
  },
  ...
}
```

--

## Keys

- Keys specify which column contains the __user_id__.
- Specify how personal tables can be joined <!-- .element: class="fragment" -->
- Specify how non-personal and personal tables can be joined <!-- .element: class="fragment" -->
- Usually (but not restricted to) foreign keys  <!-- .element: class="fragment" -->

--

![Image](content/images/data-source.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Keys - definition

- A key is a tuple: `{"<Key-Name>": "<Column-Name>"}`
- Each column can only have one key
- Multiple columns in a table can share the same `key-type`

--

## Keys example

`Purchases` table:

```json
"keys": [
  {"user_id": "UserId"},
  {"purchase_id": "PurchaseId"}
]
```

`LineItems` table:

```json
"keys": [
  {"purchase_id": "PurchaseId"},
  {"product_id": "ProductId"}
]
```

--

## Tables - inline query

`query` as an alternative to `db_name`:

```json
"tables": {
  "tablenameShownInAircloak": {
    "query": "
      SELECT cast(t2.uid as integer), t2.age, t1.*
      FROM t1 INNER JOIN t2 ON t1.pk = t2.fk
      WHERE t2.age > 18
    ",
    "content_type": "personal" | "non-personal",
    "keys": [{"key_type_1": "column_name_1"}, ...],
    ...
  },
  ...
}
```

---

## Analysis queries

- Aircloak runs background analysis queries
- A set of queries will be run against each column of a data source 
- The results of these influence the functionality and the restrictions applied

--

## Types of analysis queries

- Isolating column analysis
- Shadow database
- Bounds checks

--

## Analysis - isolating columns

- Detects if a column a quasi-identifier (i.e. on par with the user id column)
- Isolating columns underlie additional query restrictions

--

## Analysis - isolating columns - disabling

The analysis is on by default. It can be disabled on a per table basis:

```json
"tableName": {
  "auto_isolating_column_classification": false,
  ...
},
```

--

## Analysis - isolating columns - manual classification

You can manually classify whether a column is isolating or not.

```json
"tableName": {
  "isolating_columns": {"telephone_number": true, "first_name": false},
  ...
},
```

- A column that has not been manually classified will be treated as isolating if 
  automatic classification has been disabled. 
- If automatic classification is enabled a column will be treated as isolating 
  until the system has classified it.

--

## Shadow Database

- The Cloak maintains a list of the most popular terms per column
- These values are used to determine whether negative `WHERE`-clauses or `column IN (...)`
  are allowed or not

--

## Shadow Database - disabling

- The analysis can be disabled on a per-table basis

```json
"tableName": {
  "maintain_shadow_db": false,
  ...
},
```

--

## Bounds checks

- The Cloak maintains a set of anonymized minimum and maximum values per numerical column
- These are used in static analysis of queries to determine if a query construct might overflow 
  or underflow and lead to an exception in the database
- Potential over or underflow lead to Aircloak using safe (but slow) alternatives to functions

--

## Bounds checks - disabling

- Bounds analyses can be enabled or disabled on the level of a data source (not table)

```json
{
  "name": "DataSourceName",
  "driver": "oracle",
  "parameters": ...,
  "analyst_tables_enabled": true,
  "tables": ...,
  "bound_computation_enabled": false
}
```

--

## Analysis summary

- Analysis queries can be computationally expensive to run
- Aircloak attempts to run as few as possible
- We recommend enabling rather than disabling analysis queries where possible
- The system will function even with analyses queries disabled

---

## Break

---



## Session 3:
# Supporting yourself and others

Common mistakes, problems, and gotchas.

---

## "Why are all the rows anonymized away!?"

--

### Fine grained values

| UserID | BirthTime              |
|--------|-----------------------:|
| User 1 | 1980-01-01 12:15:01.02 |
| User 2 | 1980-02-01 02:21:00.12 |
| User 3 | 1980-02-01 02:50:05.99 |
| ...    | ...                    |
| User N | 1980-12-30 18:30:05.19 |

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

- Identifying information is suppressed
- Aircloak cannot tell you what this data was
- We give you an anonymizing estimate of how much data was suppressed

--

## Accounting for effects of anonymization - `*`

![Image](content/images/star-row.png) <!-- .element: style="max-height:600px;border:none;" -->

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

The difference reveals whether or not the inhabitant of the apartment is in our data set or not.

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

### Low count filter would suppress these

![Image](content/images/salaries-2.png) <!-- .element: style="max-height:600px;border:none;" -->
<!-- -- data-transition="none" -->

--

### Aggregates are different

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

### Add Gaussian noise

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

- Enough data to reveal that "Car" exists in the data set
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

> This property exists in my data set. But I CANNOT make any statistical assessment of any aggregate properties

--

### count = 2?

The SQL standard doesn't allow `null` as a return value for `count`. We would never report something that was not at the _very least_ shared by two or more users.

Read a count of 2 as meaning: 

> This property exists in my data set. I DO NOT know for how many users.

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

---

# Concepts and configuration

--

## Authentication and Authorization

--

## LDAP vs local users

--

## App logins and API tokens

--

## Debug exports

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

## CTAS

- How do they work
- What is the impact of usage

---

## Configuring Insights Air

- Docs is a good source for looking up configuration toggles
- Single json config-file

--

## Insights Air config 

```
{
  "name": string,
  "database": ...,
  "site": ...,
  "psql_server": ...,
  "ldap": ...
}
```

--

## Insights Air - name

The name allows multiple distinct installations to manage CTAS tables without conflict.

--

## Insight Air - database

Connection parameters to the Postgres database used for:

- meta data 
- user accounts (even when LDAP is used!)
- groups and group assignments
- audit log
- anonymized query results

--

## Insight Air - site

```json
"site": {
  ...
  "cloak_secret": secret_string,
  "privacy_policy_file": string,
  "license_file": string,
  "users_and_datasources_file": string,
  "browser_long_polling": boolean
},
```

Allows automatic deployment of pre-configured systems.

--

## Insight Air - psql_server

- Not to be confused with the `database` parameter
- Specifies how the built in "Postgres server" behaves

--

## Insight Air - ldap

Enables and configures the LDAP connection

---

## Insights Cloak config

```json
{
  "air_site": string,
  "salt": string,
  "cloak_secret": string,
  "data_sources": string,
  "lcf_buckets_aggregation_limit": integer,
  "max_parallel_queries": positive_integer,
  "connection_keep_time": integer,
  "connection_timeout": integer
}
```

---

## Aircloak JIRA

---

# Q&A