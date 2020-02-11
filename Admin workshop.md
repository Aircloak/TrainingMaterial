# Admin training

The workshop is meant to be 3 hour long(?).
Each session taking 45 minutes, allowing for 15 minute
smoking and coffee breaks between sessions.

The rough outline of the course is:

- What is Aircloak and what are it's parts
- Data source configuration
- System configuration
- Supporting users and yourselves (common mistakes, problems, and gotcha's)

# Session outline

## Session 1 – What is Aircloak and what are it's parts

- Agenda
- What is Aircloak
  - Why it exists
  - What it does
- Components
  - Air (docker image)
  - Cloak (docker image)
  - Air DB (external Postgres)
    - What does it store?
    - How is it maintained?
  - Data source as an abstract construct
  - LDAP (external service)
- Scaling
  - Cloak to data source mapping
  - Effects of multiple cloaks
    - Each query associated with one cloak
      - Parallelism at level of query
- _TEF_: Installation at Telefonica
  - Infrastructure
    - Kubernetes
    - Postgres instance
    - LDAP
  - How is it maintained
    - Component configuration maintenance
    - Data source definitions
    - Upgrades and deployments
  - People

## Session 2 – Data source configuration

- What is a data source
- One file per data source
  - JSON format
  - Parts
    - Name
    - Data source connector type
    - Connection parameters
    - Tables and how to anonymize
- Personal vs non-personal tables
  - What is a non-personal table
  - Why and when does a non-personal table need keys
- Anonymization based on user-id
  - What is a user-id and how does it impact anonymization
  - When does Aircloak anonymize
  - How to choose a user-id:
    - What are good/bad candidates
  - Types of user-ids at TEF
    - ClientID, PartyID, ContractID vs VendorID
- Tables
  - How to add/remove
  - Content type
  - DB table vs query
  - Aircloak name vs DB name
  - Keys
    - Special key: `user_id`
    - keys to column
      - 1 key-type per column
      - multiple columns can share the same key-type
  - Analyses
    - What are they?
    - Which forms exist?
    - Enable/disable
    - Manual classification
- Quizzes

## Session 3 – Supporting users and yourselves (common mistakes, problems, and gotcha's)

- Admin interface
  - How to access
  - What are the parts
- User guide
- Users
  - LDAP vs local users
    - Admin only as local users
    - Disabling user account vs deleting
  - Resetting passwords
    - Command line for admin
    - LDAP is separate procedure
    - Local users in admin interface
- Group based permissions
  - LDAP vs local
- Common pitfalls
  - `SELECT *`
  - `SELECT ... LIMIT X` or `SELECT ... WHERE ROWNUM < X`
  - Query containing unique columns
  - High dimensional queries
  - High resolution data: generalization
- Debug Exports
  - How to enable
  - How to access
  - How to read
- CTAS
  - Enabled on a per data source level
  - Takes storage in Oracle
  - Can be viewed in the admin interface
- App logins
- API tokens
- System health monitoring

## Session 4 – System configuration and maintenance

- License
- Air configuration options
  - Auto configuration
- Cloak configuration options
  - Cloak level toggles
- How to upgrade
  - Air auto-migrates DB (take backup first)
  - Stage environment
    - Separate installation
    - Stage data sources
- Logs
- _TEF_:
  - User management (JIRA)
  - Data source management process
  - Data protection process

# Auxiliary

Google docs with original outlines by Arnold as well as some notes can be found here:
https://docs.google.com/document/d/1xNMRerWwILcOW_WXBSXqWor95K8Kgk6N_JANuFeDmkc/edit#
