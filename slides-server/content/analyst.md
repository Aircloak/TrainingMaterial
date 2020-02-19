# Analyst training
## Aircloak Insights

Telefonica – 2020

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

# Introductions

--

## We are

- Felix (legal, thought leadership)
- Sebastian (tech)

--

## You are

Quick round. What is your:
- name
- role

---

Before we start

# What is Aircloak?

--

## Aircloak 

- is an SQL query proxy <!-- .element: class="fragment"-->
- performs dynamic anonymization at runtime <!-- .element: class="fragment"-->
- no risk of accidental data disclosure <!-- .element: class="fragment"-->

--

## It is accessible through ...

--

<!-- .slide: data-transition="none" data-background="content/images/aircloak-interface.png" -->

## a browser interface <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


--

<!-- .slide: data-transition="none" data-background="content/images/rest-api.png" -->

## a REST API <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


--

<!-- .slide: data-transition="none" data-background="content/images/psql.png" -->

## Postgres query interface <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->

---


## Session 1: 
# Anonymization

---

## Terms and how they differ

- Pseudonymization
- Data masking
- Anonymization

--

### Raw data

| UserID | Age | Salary | Name    |
|--------|----:|-------:|---------|
| 1      | 10  |     0  | Alice   |
| 2      | 20  | 20000  | Bob     |
| 3      | 30  | 50000  | Cynthia |

--

### Pseudonymized data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| A      | 10  |     0  | Alvar   |
| B      | 20  | 20000  | Borg    |
| C      | 30  | 50000  | Crantor |

--

### Masked data - partial

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| 1      | 10  |     X  | Xlice   |
| 2      | 20  | 20XXX  | Xob     |
| 3      | 30  | 50XXX  | Xynthia |

--

### Masked data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| 1      |  10 |     X  | X       |
| 2      |  20 |     X  | X       |
| 3      |  30 |     X  | X       |

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

- Both pseudonymous and data masking yields per user data
- Despite masking and/or redaction inference is often possible with (and sometimes without) additional knowledge

--

### Confusing these categories leads to disaster

CUE: Felix tell stories

---

## Anonymization and Aircloak

---

### Per-user anonymization

--


### Example

#### Scenario

We have the knowledge that Alfred has bought an **obscure product**.

--

These are the records of all purchases of obscure products: 

| Product         |
|-----------------|
| Obscure product |
| Obscure product |
| Obscure product |
| Obscure product |
| Obscure product |

Question: Does this dataset give us additional knowledge about Alfred? <!-- .element: class="fragment" -->

Answer: hard to say... <!-- .element: class="fragment" -->

--

| Product         | Additional info |
|-----------------|-----------------|
| Obscure product | red hair        |
| Obscure product | has cancer      |
| Obscure product | unfaithful      |
| Obscure product | has bad breath  |
| Obscure product | no info         |

Question: Does this dataset give us additional knowledge about Alfred? <!-- .element: class="fragment" -->

Answer: again it's hard to say... <!-- .element: class="fragment" -->

--

| UserID | Product         | Additional info |
|--------|-----------------|-----------------|
| User 1 | Obscure product | red hair        |
| User 2 | Obscure product | has cancer      |
| User 3 | Obscure product | unfaithful      |
| User 4 | Obscure product | has bad breath  |
| User 5 | Obscure product | no info         |

Question: and now? <!-- .element: class="fragment" -->

Answer: Alfred could be any of these people... <!-- .element: class="fragment" -->

--

| UserID | Product         | Additional info | Year | 
|--------|-----------------|-----------------|------|
| User 1 | Obscure product | red hair        | 2016 |
| User 2 | Obscure product | has cancer      | 2017 |
| User 3 | Obscure product | unfaithful      | 2018 |
| User 4 | Obscure product | has bad breath  | 2019 |
| User 5 | Obscure product | no info         | 2020 |

Question: and now? <!-- .element: class="fragment" -->

Answer: this might very likely allow us to deduce who Alfred is! <!-- .element: class="fragment" -->

--

| UserID | Product         | Additional info | Year | 
|--------|-----------------|-----------------|------|
| User 1 | Obscure product | red hair        | 2016 |
| User 1 | Obscure product | has cancer      | 2017 |
| User 1 | Obscure product | unfaithful      | 2018 |
| User 1 | Obscure product | has bad breath  | 2019 |
| User 1 | Obscure product | no info         | 2020 |

Question: and now? <!-- .element: class="fragment" -->

Answer: now Alfred is very easy to identify! <!-- .element: class="fragment" -->

--

## Morale of the story

- External information is dangerous <!-- .element: class="fragment" -->
- It's not the number of purchas**es** that determine identifiability. It's the number of purchas**ers** <!-- .element: class="fragment" -->
- Aircloak does per user anonymization <!-- .element: class="fragment" -->

---

## Low count filtering

--

### In summary

Information shared by enough distinct users is generally safe. It gets through the filter.

"Enough" means ~5 (more later) <!-- .element: class="fragment" -->

--

## Quiz

--

### Quiz 1 
#### Is "Obscure product" ok to reveal? 

| UserID | Product         |
|--------|-----------------|
| User 1 | Obscure product |
| User 2 | Obscure product |
| User 3 | Obscure product |
| User 4 | Obscure product |
| User 5 | Obscure product |
| User 6 | Obscure product |

Answer: **Yes. ~5 users share the property!** <!-- .element: class="fragment" -->

--

### Quiz 2
#### Is "Obscure product" ok to reveal? 

| UserID | Product         |
|--------|-----------------|
| User 1 | Obscure product |
| User 1 | Obscure product |
| User 1 | Obscure product |
| User 1 | Obscure product |
| User 1 | Obscure product |
| User 1 | Obscure product |

Answer: **No. Only a single user share the property!** <!-- .element: class="fragment" -->

--

### Example: fine grained values

--

### Example: fine grained values

| UserID | BirthTime              |
|--------|-----------------------:|
| User 1 | 1980-01-01 12:15:01.02 |
| User 2 | 1980-02-01 02:21:00.12 |
| User 3 | 1980-02-01 02:50:05.99 |
| ...    | ...                    |
| User N | 1980-12-30 18:30:05.19 |

--

![Image](content/images/lcf-g1.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/lcf-g2.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/lcf-g3.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/lcf-g4.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/lcf-g5.png) <!-- .element: style="max-height:600px;border:none;" -->

--

![Image](content/images/lcf-g6.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## But, now, what happens if we add in say the gender?

--

![Image](content/images/lcf-g7.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## But!

Gender and birthday cannot be that identifying!?

There are thousands of people born every day!

--

## There is a hidden attribute!

--

## You think of your data as

| Gender | BirthTime              |
|--------|-----------------------:|
| M      | 1980-01-01 12:15:01.02 |
| M      | 1980-02-01 02:21:00.12 |
| F      | 1980-02-01 02:50:05.99 |
| ...    | ...                    |
| F      | 1980-12-30 18:30:05.19 |

--

## But in reality it is

| Gender | BirthTime              | Is our customer |
|--------|-----------------------:|----------------:|
| M      | 1980-01-01 12:15:01.02 | True            | 
| M      | 1980-02-01 02:21:00.12 | True            |
| F      | 1980-02-01 02:50:05.99 | True            |
| ...    | ...                    | True            |
| F      | 1980-12-30 18:30:05.19 | True            |

--

## This implicit attribute in turn makes all the other data more identifying.

--

| Home address | Is our customer | Count |
|--------------|----------------:|------:|
| Human St.    | True            | 20    |
| Human St.    | False           | 250   |
| Albert Av.   | True            | 50    |
| Albert Av.   | False           | 520   |
| My corp Av.  | True            | 204   |
| My corp Av.  | False           | 1     |

--

| Home address | Age | Is our customer | Count |
|--------------|----:|----------------:|------:|
| Human St.    | 1   | True            | 0     |
| Human St.    | 1   | False           | 5     |
| Human St.    | 2   | True            | 1     |
| Human St.    | 2   | False           | 15    |
| ...          | ... | ...             | ...   |

--

# In summary

> Values (or sets of values) can only be revealed if enough _distinct users_ share them

---

# Aggregate statistics

--

## Sample `purchases` table

| UserID | Product   | Price   |
|--------|-----------|--------:|
| User 1 | Product A | 10      |
| User 2 | Product B | 20      |
| User 3 | Product A | 10000   |
| User 4 | Product C | 30      |
| User 5 | Product B | 0       |

--

## count

```sql
SELECT count(*)
FROM products
```

- The real count is 5
- Aircloak anonymizes and might return 6

--

## count + count_noise

```sql
SELECT count(*), count_noise(*)
FROM products
```

- `count_noise(X)` returns the _approximate_ standard deviation used when adding noise to `count(X)`

--

## sum + sum_noise

```sql
SELECT sum(price), sum_noise(price)
FROM products
```

- Produces an anonymous sum where `sum_noise` is the _approximate_ standard deviation of the noise

--

## avg + avg_noise

```sql
SELECT avg(price), avg_noise(price)
FROM products
```

- Produces an anonymous average where `avg_noise` is the _approximate_ standard deviation of the noise

--

## stddev + stddev_noise

```sql
SELECT stddev(price), stddev_noise(price)
FROM products
```

- Produces an anonymous standard deviation where `stddev_noise` is the _approximate_ standard deviation of the noise

--

## Why the noise is approximate

- The noise attempts to masque the impact of the most extreme users
- The suppression of an extreme value should not be visible in the `_noise` value

--

## Consider the data

__User 3__ paid  1000 times more for Product A than did __User 1__.

| UserID | Product   | Price   |
|--------|-----------|--------:|
| User 1 | Product A | 10      |
| User 2 | Product B | 20      |
| User 3 | Product A | 10000   |
| User 4 | Product C | 30      |
| User 5 | Product B | 0       |

--

## Effect of an extreme user

| Aggregate | Including | Excluding |
|-----------|----------:|----------:|
| count     | 5         | 4         |
| sum       | 10060     | 60        |
| average   | 2012      | 15        |
| stddev    | 4465      | 13        |

Aircloak makes your analytics look more like the extreme user was not part of the results.

--

## Effect of an extreme user

| Aggregate | Including | Excluding | Delta |
|-----------|----------:|----------:|------:|
| count     | 5         | 4         | 1     |
| sum       | 10060     | 60        | 10000 |
| average   | 2012      | 15        | 1997  |
| stddev    | 4465      | 13        | 4452  |

The effective distortion can be large. 

If `count_noise`, `sum_noise`, `avg_noise`, or `stddev_noise` would expose the difference then you could easily learn about  __User 3__.

--

## "Attack" example

We know that "User 3" always pays too much. How do we figure out which products he bought?

```sql
SELECT product, stddev(price)
FROM purchases
GROUP BY product
ORDER BY stddev(price) DESC
```

--

## What about `min_noise` and `max_noise`?

They do not exist. No sensible value exists that could be reported. Just like how the other `_noise` functions cannot tell you about the extreme values they suppress.

---

## User IDs selection

- Aircloak anonymizes based on users.
- It needs to be able to associate a user id value to each row in a database

--

![Image](content/images/data-source.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Types of tables

- **personal**: contains personal data
- **non-personal**: does not contain personal data. Typically fact or lookup tables

--

## Types of tables

- **personal**: require anonymization
- **non-personal**: usually do not require anonymization

--

## Recap: every query must reference a table with a user-id column!

--

## Quiz – query 1: Ok?

```sql
SELECT name, count(*)
FROM users
```

Answer:
<span class="fragment">
Yes. `users` has a `UserId`-column 
</span>

--

## Quiz – query 2: Ok?

```sql
SELECT count(*)
FROM purchases
```

Answer:
<span class="fragment">
Yes. `purchases` has a `UserId`-column 
</span>

--

## Quiz – query 3: Ok?

```sql
SELECT count(*)
FROM lineItems
```

Answer:
<span class="fragment">
No. There is no `UserId`-column in the `lineItems` table.
</span>

--

## Query 3 solution:

Include a `JOIN` to a table with a `UserId` column:

```sql
SELECT count(*)
FROM lineItems inner join purchases
  ON lineItems.purchaseId = purchases.purchaseId
```

--

## Exception: lookup/fact tables

Tables that do not contain any personal information (such as the `products` table) can be queried alone without a `UserId` column being present.

The reason: the query does not need anynimyzation:

```sql
SELECT productName
FROM products
```

--

## Unless...

If lookup tables are used _together_ with tables containing personal data, then the normal rules apply:

This is forbidden:

```sql
SELECT productName, count(*)
FROM products INNER JOIN lineItems
  ON products.productId = lineItems.productId
```

Whereas this is allowed:

```sql
SELECT productName, count(*)
FROM products INNER JOIN lineItems
  ON products.productId = lineItems.productId
  INNER JOIN purchases 
  ON lineItems.purchaseId = purchases.purchaseId 
```

--

## Data model refresher

![Image](content/images/data-source.png) <!-- .element: style="max-height:600px;border:none;" -->

---

## Types of queries

- Anonymizing queries
- Restricted queries
- Unrestricted queries

--

## Anonymizing queries

- Any query that **aggregates or groups** across multiple distinct users

```sql
SELECT name, count(*)
FROM users
GROUP BY name
```

--

## Restricted query

- Any query that operates on the data of a single user

```sql
SELECT UserId, count(*) as numPurchases
FROM purchases
GROUP BY UserId
```

--

## Unrestricted query - 1

- A query that operates on non-personal data 
- A query that operates on an already anonymous result

```sql
SELECT productName
FROM products -- lookup table
```

--

## Let's combine them!

--

## A restricted query:

```sql
-- Restricted query
SELECT userId, productId, count(*) as purchasedByUser
FROM lineItems li INNER JOIN purchases p
  ON li.purchaseId = p.purchaseId
GROUP BY userId, productId
```

--

## ... within an anonymizing query

```sql
-- Anonymizing query
SELECT 
  productId, 
  purchasedByUser, 
  count(*) as usersWithNumPurchases
FROM (
  -- Restricted query
  SELECT userId, productId, count(*) as purchasedByUser
  FROM lineItems li INNER JOIN purchases p
    ON li.purchaseId = p.purchaseId
  GROUP BY userId, productId
) as perUserProductPurchases
```

--

## ... within an unrestricted query!

```sql
-- Unrestricted query
SELECT productId
FROM (
  -- Anonymizing query
  SELECT 
    productId, 
    purchasedByUser, 
    count(*) as usersWithNumPurchases
  FROM (
    -- Restricted query
    SELECT userId, productId, count(*) as purchasedByUser
    FROM lineItems li INNER JOIN purchases p
      ON li.purchaseId = p.purchaseId
    GROUP BY userId, productId
  ) as perUserProductPurchases
) as perUserProductPurchaseStats
HAVING SUM(purchasedByUser) * SUM(usersWithNumPurchases) > 100
```

--

## Same query – less complex

```sql
SELECT productId, count(*) as timesPurchased
FROM lineItems li INNER JOIN purchases p
  ON li.purchaseId = p.purchaseId
GROUP BY productId
HAVING timesPurchased > 100
```

__Quiz__: what query type is this?

--

## Data model refresher

![Image](content/images/data-source.png) <!-- .element: style="max-height:600px;border:none;" -->

--

## Final gotcha

 What query type is this?

 ```sql
SELECT purchaseId
FROM purchases
 ```

 <span class="fragment">
 <strong>Answer:</strong> restricted but not anonymizing. There is no cross user grouping. One input row to one output row and there is a User id column in the table. 
 </span>

--

## End with a simpler one

 What query type is this?

 ```sql
SELECT count(*)
FROM users
 ```

 <span class="fragment">
 <strong>Answer:</strong> anonymizing. Cross user aggregation (which is a grouping)
 </span>