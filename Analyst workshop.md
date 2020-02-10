# Analyst training

The workshop is meant to be 4 hour long.
Each session taking 45 minutes, allowing for a 15 minute
smoking and coffee break between sessions. We anticipate
that 4 hours will be enough to get the users up to speed,
but hopefully not so long that we loose their attention.

Each session is meant to be thematically self-contained.
Ideally they should be rather interactive too.

The rough outline of the course is:

1. Intro and info about anonymization
2. SQL details and nitty gritty details of Aircloak
3. Attempt at independent problem solving, with a price
4. Practical installation specific information

The fourth session is the one where the most customer
specific information is envisioned. It should also be
the only one that needs to be adapted per customer.

# Session outline

## Session 1

- Agenda – what will we be doing?
- Introduction
  - quick round around the room, names + roles
- What is Aircloak?
- What is anonymization?
  vs pseudonymization
  vs data masking
- Anonymization and Aircloak
  - Low count filtering
  - Effect of high dimensionality
    - Implicit dimension [isCustomer = true] makes even non-sensitive looking tuples like `(surname, age)` sensitive
    - Aggregation and related suppression
      - Noise functions and their limitations

## Session 2

- `SELECT *`
- `SELECT ... LIMIT 10`
- Quick overview over functionality
  - Point to docs capabilities slide
  - Show how to find the "User guides"
- Aggregates
  - `null`-values
    - When, why
  - `_noise`
    - must be used the same way
    - how it can be used with `HAVING` for data quality
- Missing data
  - noticing the presence of `*`
    - what does it mean? (i.e. semantic is aggregate dependent)
  - regrouping with: `bucket`, `date_trunc`, `left`/`substring`
- Working with free form text
- User id selection
  - keys
  - query types: unrestricted, restrited, anonymizing
    - Quizzes to hammer in the different query types
- Building co-horts using sub-queries
- Using Views to reuse logic
- Use CTAS to speed up queries
- Restrictions
  - Limited OR-functionality
  - Ranges
  - other...

## Session 3

This is supposed to be a practical session.
Unclear what the goal is.
Should be sufficiently complex that it requires multiple
queries to get to the solution, and such that it involves
the different aspects we have learnt.

Questions:

- How do we determine who is the winner?
- What is the problem they are trying to solve?

## Session 4 – Aircloak and you (practical details)

- Working around missing features with views in Oracle
- How to get access to a data source: Jira ticket
- Internal handbook with Telefonica specific gotchas and learnings
- How to find help (the docs)
  – 1st level support at Telefonica, 2nd level support at Aircloak
- PSQL interface
  - Query from python
  - Oracle DB Link / DB Gateway

# Outline of Analyst topics by Arnold

See https://docs.google.com/document/d/1xNMRerWwILcOW_WXBSXqWor95K8Kgk6N_JANuFeDmkc/edit#

b. Users

1.                            What is Aircloak
2.                            Anonymity
3.                            What is anonymity?
4.                            How does Aircloak define / implement it?
5.                            Consequences & Effects & Restrictions
    a. NOISE -> NOISE-functions
    b. Hidden data -> no way to find out
    c. KPI Outliers
    d. k-Anonymity
    e. No data persistence during Aircloaks operations
6.                            How to get access? -> TEF
7.                            Web
8.                            PostgreSQL
9.                            TEF process
10.       Querying
11.       User-ids in depth
12.       Process
    a. Aircloak-Query
    b. Database
    c. Ingest
    d. Process
13.       Aircloak SQL
    a. Syntax
    b. Available clauses: SELECT, FROM, WHERE, GROUP BY, HAVING
    c. Scalar-Functions overview
    d. Aggregation
    e. Kinds of Queries
    i. Standard-Queries
    ii. Restricted-Queries
    iii. Anonymizing-Queries
    f. Date arithmetics
    g. Difference to ORACLE SQL / Current shortcomings
    i. Syntax
    ii. No analytic functions
    iii. DATE range
14.       How to read results? Understanding results.
    a. COUNTS(*) == 2
    b. Differences to unprotected queries?
    c. Hidden data
    d. *NOISE functions
15.       Performance, accuracy
    a. HAVING is processed in Aircloak
    b. Aggregate as much as possible in ORACLE
16.       Aircloak Views and Aircloak tables (CTAS)
17.       Workarounds for missing features
    a. How to define views which can be included?
    b. How to have feature x…?
18.       How to access results
19.       Csv
20.       History
21.       DB-Link (if available)
22.       Postgres-Port
23.       PL/SQL example (only if DB-Link available)
24.       Python scripting example
25.       Extending data source
26.       Which tables can be integrated?
27.       How -> TEF process
28.       Support-process -> TEF
