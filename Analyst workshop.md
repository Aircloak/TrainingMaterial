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

- delayed start ... [5 minutes]
- Welcome and agenda – what will we be doing? – [2 minutes]
- Introduction - [5 minutes]
  - quick round around the room, names + roles
- What is Aircloak? [3 minutes]
  - Query proxy
  - Accessible through web interface, and Postgres interface
- What is anonymization? [5 minutes]
  vs pseudonymization
  vs data masking
  cautionary tales
- Anonymization and Aircloak [13 minutes]
  - "Per-user"-anonymization
  - Low count filtering
    - Sometimes you will know, othertimes you will not
  - Effect of high dimensionality
    - Implicit dimension [isCustomer = true] makes even non-sensitive looking tuples like `(surname, age)` sensitive
    - Aggregation and related suppression
      - Outlier suppression
      - Noise functions and their limitations
      - Suppression we cannot tell you about
- User id selection [12 minutes]
  - keys
  - query types: unrestricted, restrited, anonymizing
    - Quizzes to hammer in the different query types

## Session 2

- Quick overview over functionality [10 minutes]
  - Point to docs capabilities slide
  - Show how to find the "User guides"
  - Differences to Oracle SQL
    - No WINDOW function support
- Common pitfalls [2 minutes]
  - `SELECT *`
  - `SELECT ... LIMIT 10`
- Missing data [5 minutes]
  - noticing the presence of `*`
    - what does it mean? (i.e. semantic is aggregate dependent)
  - regrouping with: `bucket`, `date_trunc`, `left`/`substring`
- Aggregates [7 minutes]
  - `null`-values
    - When, why
  - `_noise`
    - must be used the same way
    - how it can be used with `HAVING` for data quality
    - the noise we cannot tell you about
- Working with free form text [3 minutes]
- Building co-horts using sub-queries [4 minutes]
- Using Views to reuse logic [2 minutes]
- Use CTAS to speed up queries [2 minutes]
- Restrictions [10 minutes]
  - Limited OR-functionality
  - Ranges
    - How to work with them
    - Date ranges
  - other...

## Session 3

- Explain data model [2 minutes]
- Explain task [3 minutes]
- Explain how they get access [5 minutes]
- Let them work [20 minutes]
- Answer questions or explain how they solved it [15 minutes]

* Using BlaBla account
  This is supposed to be a practical session.
  Unclear what the goal is.
  Should be sufficiently complex that it requires multiple
  queries to get to the solution, and such that it involves
  the different aspects we have learnt.

This session is very much dependent on the data we have
available for analysis. We should create an interesting
use case around the particular data.

Questions:

- How do we determine who is the winner?
- What is the problem they are trying to solve?

## Session 4 – Aircloak and you (practical details)

- TEF: How does one access?
  - Web interface (as used throughout)
  - PSQL interface
    - Show python example code
    - Oracle DB Link / DB Gateway
  - Web API (can probably skip this...)
- TEF: How to get access to a data source: Jira ticket
- TEF: How to add new data
- TEF: Working around missing features with views in Oracle
- Internal handbook with Telefonica specific gotchas and learnings
- How to find help (the docs)
  – 1st level support at Telefonica, 2nd level support at Aircloak
  - Debug exports / Enable debug mode
- Q/A and play around and ask questions if you get stuck

# Auxiliary

Google docs with original outlines by Arnold as well as some notes can be found here:
https://docs.google.com/document/d/1xNMRerWwILcOW_WXBSXqWor95K8Kgk6N_JANuFeDmkc/edit#
