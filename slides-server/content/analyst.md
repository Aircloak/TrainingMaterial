# Analyst training
## Aircloak Insights

Telefonica – 2020

---

# Agenda

1. Anonymization
1. Querying with Aircloak
1. Practical session
1. Telefonica specific details

>>>

## Sessions

- Start on the hour
- Each session takes ~45 minutes
- 15 minute break between sessions

---

# Introductions

>>>

## We are

- Felix (legal, thought leadership)
- Sebastian (tech)

>>>

## You are

Quick round. What is your:
- name
- role

---

Before we start

# What is Aircloak?

>>>

## Aircloak 

- is an SQL query proxy <!-- .element: class="fragment"-->
- performs dynamic anonymization at runtime <!-- .element: class="fragment"-->
- no risk of accidental data disclosure <!-- .element: class="fragment"-->

>>>

## It is accessible through ...

>>>

<!-- .slide: data-transition="none" data-background="content/images/aircloak-interface.png" -->

## a browser interface <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


>>>

<!-- .slide: data-transition="none" data-background="content/images/rest-api.png" -->

## a REST API <!-- .element: style="position: absolute; width: 40%; right: 0; box-shadow: 0 1px 4px rgba(0,0,0,0.5), 0 5px 25px rgba(0,0,0,0.2); background-color: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px; font-size: 20px; text-align: left;" -->


>>>

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

>>>

### Raw data

| UserID | Age | Salary | Name    |
|--------|----:|-------:|---------|
| 1      | 10  |     0  | Alice   |
| 2      | 20  | 20000  | Bob     |
| 3      | 30  | 50000  | Cynthia |

>>>

### Pseudonymized data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| A      | 10  |     0  | Alvar   |
| B      | 20  | 20000  | Borg    |
| C      | 30  | 50000  | Crantor |

>>>

### Masked data - partial

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| 1      | 10  |     X  | Xlice   |
| 2      | 20  | 20XXX  | Xob     |
| 3      | 30  | 50XXX  | Xynthia |

>>>

### Masked data

| UserID | Age | Salary | Name    | 
|--------|----:|-------:|---------|
| 1      |  10 |     X  | X       |
| 2      |  20 |     X  | X       |
| 3      |  30 |     X  | X       |

>>>

### Anonymization

In this particular example: 

> There were roughly `3` users in the dataset

>>>

### Anonymization

More generally some form of aggregation and distortion:

| Age | Salary | Name | Noisy count | 
|----:|-------:|------|------------:|
| 10  |     0  | *    |         250 |
| 12  |     0  | *    |         201 |
| 13  |  1-200 | *    |          43 |
| ... |    ... | *    |         ... |
| 84  | 200000 | *    |          14 |

>>>

### Notice:

- Both pseudonymous and data masking yields per user data
- Despite masking and/or redaction inference is often possible with (and sometimes without) additional knowledge

>>>

### Confusing these categories leads to disaster

CUE: Felix tell stories

---

## Anonymization and Aircloak

---

### Per-user anonymization

>>>


### Example

#### Scenario

We have the knowledge that Alfred has bought an **obscure product**.

>>>

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

>>>

| Product         | Additional info |
|-----------------|-----------------|
| Obscure product | red hair        |
| Obscure product | has cancer      |
| Obscure product | unfaithful      |
| Obscure product | has bad breath  |
| Obscure product | no info         |

Question: Does this dataset give us additional knowledge about Alfred? <!-- .element: class="fragment" -->

Answer: again it's hard to say... <!-- .element: class="fragment" -->

>>>

| UserID | Product         | Additional info |
|--------|-----------------|-----------------|
| User 1 | Obscure product | red hair        |
| User 2 | Obscure product | has cancer      |
| User 3 | Obscure product | unfaithful      |
| User 4 | Obscure product | has bad breath  |
| User 5 | Obscure product | no info         |

Question: and now? <!-- .element: class="fragment" -->

Answer: Alfred could be any of these people... <!-- .element: class="fragment" -->

>>>

| UserID | Product         | Additional info | Year | 
|--------|-----------------|-----------------|------|
| User 1 | Obscure product | red hair        | 2016 |
| User 2 | Obscure product | has cancer      | 2017 |
| User 3 | Obscure product | unfaithful      | 2018 |
| User 4 | Obscure product | has bad breath  | 2019 |
| User 5 | Obscure product | no info         | 2020 |

Question: and now? <!-- .element: class="fragment" -->

Answer: this might very likely allow us to deduce who Alfred is! <!-- .element: class="fragment" -->

>>>

| UserID | Product         | Additional info | Year | 
|--------|-----------------|-----------------|------|
| User 1 | Obscure product | red hair        | 2016 |
| User 1 | Obscure product | has cancer      | 2017 |
| User 1 | Obscure product | unfaithful      | 2018 |
| User 1 | Obscure product | has bad breath  | 2019 |
| User 1 | Obscure product | no info         | 2020 |

Question: and now? <!-- .element: class="fragment" -->

Answer: now Alfred is very easy to identify! <!-- .element: class="fragment" -->

>>>

## Morale of the story

- External information is dangerous <!-- .element: class="fragment" -->
- It's not the number of purchas**es** that determine identifiability. It's the number of purchas**ers** <!-- .element: class="fragment" -->
- Aircloak does per user anonymization <!-- .element: class="fragment" -->

---

## Low count filtering

>>>

### In summary

Information shared by enough users gets through

Enough means ~5 (more later) <!-- .element: class="fragment" -->

>>>

## Quiz

>>>

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

>>>

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