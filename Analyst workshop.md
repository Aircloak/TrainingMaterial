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
  - Query proxy
  - Accessible through web interface, and Postgres interface
- What is anonymization?
  vs pseudonymization
  vs data masking
- Anonymization and Aircloak
  - "Per-user"-anonymization
  - Low count filtering
    - Sometimes you will know, othertimes you will not
  - Effect of high dimensionality
    - Implicit dimension [isCustomer = true] makes even non-sensitive looking tuples like `(surname, age)` sensitive
    - Aggregation and related suppression
      - Outlier suppression
      - Noise functions and their limitations
      - Suppression we cannot tell you about

## Session 2

- `SELECT *`
- `SELECT ... LIMIT 10`
- Quick overview over functionality
  - Point to docs capabilities slide
  - Show how to find the "User guides"
  - Differences to Oracle SQL
    - No WINDOW function support
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
    - How to work with them
    - Date ranges
  - other...

## Session 3

- Explain how they get access
  - Using BlaBla account
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

- TEF: How to get access to a data source: Jira ticket
- TEF: How to add new data
- How does one access?
  - Web interface (as used throughout)
  - PSQL interface
    - Show python example code
    - Oracle DB Link / DB Gateway
  - Web API
- Debug exports / Enable debug mode
- Working around missing features with views in Oracle
- Internal handbook with Telefonica specific gotchas and learnings
- How to find help (the docs)
  – 1st level support at Telefonica, 2nd level support at Aircloak
