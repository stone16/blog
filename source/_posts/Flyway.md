---
title: Flyway
date: 2022-01-01 21:57:09
categories: BackEnd 
tags:
    - database 
    - migration 
top:
---
# 1. Overview

- An open source database migration tool
- Favors simplicity and convention over configuration
- has 7 basic commands
    - migrate
    - clean
    - info
    - validate
    - undo
    - baseline
    - repair

## 1.1 Why DB migrations?

- we need a way to version the table
- we need to know what state is the db on this machine
- database migration help us
    - recreate a database from scratch
    - make it clear at all times what state a database is in
    - migrate in a deterministic way from your current version of the database to a newer one

## 1.2 How Flyway works?

- Flyway first try to locate its schema history table
    - if db empty, then flyway will create it instead
    - this default db named as `flyway_schema_history`
- Then flyway will begin scanning the filesystem or the classpath of the application for migrations
- The migrations are then sorted based on the version number and applied in order
- The schema history table will be updated accordingly as each migration gets applied

- we use `flyway migrate` to execute the migration

# 2. Flyway Commands

## 2.1 `migrate`

- help migrate the db

## 2.2 `clean`

- drop all objects in the confgured schemas

## 2.3 `info`

- print the details and status information about all migrations

## 2.4 `validate`

- validates the applied migrations against the ones available on the classpath

## 2.5 `undo`

- undoes the most recently applied versioned migration

## 2.6 `baseline`

- baselines an existing database, excluding all migrations up and including baseline version

## 2.7 `repair`

- repair the schema history table

# 3. Concepts

## 3.1 Migrations

- all changes to the db are called migrations
- migrations can be
    - versioned
        - types
            - regular
            - undo
                - the effect can be undone by supplying an undo migration
        - contains
            - **version**
                - must be unique
            - **description**
                - purely informative for you to be able to remember what each migration does
            - **checksum**
                - detect accidental changes
    - repeatable
        - contains
            - **description**
            - **checksum**
        - instead of being run just once, they are re-applied every time their checksum changes
        - Within a single migration run, repeatable migrations are always **applied last**, after all pending versioned migrations have been executed. Repeatable migrations are applied in the order of their description

### 3.1.1 Versioned Migrations

- contains
    - version
        - **must be unique**
        - applied in order exactly once
    - description
        - purely informative for you to be able to remember what each migration does
    - checksum
        - detect accidental changes
- used for
    - creating/ altering/ dropping tables/ indexes/ foreign keys/ enums
    - reference data updates
    - user data corrections

### 3.1.2 Undo Migrations

- A migration can fail at any point. If you have 10 statements, it is possible for the 1st, the 5th, the 7th or the 10th to fail. There is simply no way to know in advance. In contrast, undo migrations are written to undo an entire versioned migration and will not help under such conditions.
- we should **maintain backwards compatibility** between the **DB and all versions of the code** currently deployed in production

### 3.1.3 Repeatable Migrations

- contains
    - description and a checksum, but no version
- repeatable migrations are re-applied every time their checksum changes

- Very useful for managing database objects whose definition can then simply be maintained in a single file in version control
- Repeatable migrations are always applied last, after all pending versioned migrations have been executed; always applied in the order of their description

### 3.1.4 SQL Based Migrations

- used for
    - DDL change — CREATE/ALTER/DROP statements for TABLES,VIEWS,TRIGGERS,SEQUENCES,…
    - Simple reference data changes
    - simple bulk data changes
- Naming  Patterns
    - Prefix
        - v for versioned
        - u for undo
        - r for repeatable migrations
    - version
        - with dots or underscores separate as many parts as you like
    - Separator
        - __ two underscores
    - Suffix
        - `.sql`
- Discovery
    - Flyway discover sql migrations from directories **referenced by the location property**

### 3.1.5 Script Based Migrations

- name patten
    - ``V1__execute_batch_tool.sh`
- could be used for
    - triggering execution of a 3rd party application as part of the migrations
    - cleaning up local files

### 3.1.6 Transactions

- By default, Flyway **wraps the execution of an entire migration within a single transaction**

## 3.2 Callbacks

- For the case we need to execute same action over and over again
- we could hook into its lifecycle

- there are certain keywords we could use, and invoke them during the process

[https://flywaydb.org/documentation/concepts/callbacks](https://flywaydb.org/documentation/concepts/callbacks) 

## 3.3 Error Overrides

- By default, in case an error is returned, flyway displays it with all necessary details, marks the migration as failed and automatically rolls it back if possible
- But we could change the behavior like
    - treat an error as a waring
    - treat a waring as an error
    - perform an additional action

## 3.4 Dry Runs

- Used for
    - preview changes Flyway will make to the db
    - submit the SQL statements for review
    - use Flyway to determine what needs updating,
- how it works
    - flyway sets up a read only connection to the db,
    - assesses what migrations need to run and generates a single SQL file containing all statements it would have executed in case of a regular migration run

## 3.5 Baseline Migrations

- Over the lifetime of a project, there would be tons of db objects be created/ destroyed across many migrations
    - we want to simplify with a single, cumulative migration that represents the state of db after all of those migrations have been applied without disrupting existing env

- How it works?
    - Prefixed with B followed by the version of your db they represent
    - Only used when deploying to new env
    - If used in an env where some Flyway migrations have already been applied, **baseline migrations will be ignored,** **new env will choose the latest baseline migration as the starting point**
        - every migration with a version below the latest baseline migration's version is marked as ignored
    - baseline migration are executed during the migrate process
    

# Reference

[https://flywaydb.org/documentation/](https://flywaydb.org/documentation/)