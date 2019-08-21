# 8. Define types of schemas in our databases

Date: 07/29/2016

## Context

In the course of trying to standardize how we do database development, we have had lots of discussion around schemas (much of the conversation around how oracle specifically views schemas, but the conversation *may* be relavant to other databases). This conversation has been mostly around how do do our database development, and how do we provide appropriate access to the required data.

Througout this discussion (mostly happening in our DB Working Group), we have agreed on a set of definitions for different types of schemas.

Here are the notes from the original discussion:

## Decision

We will use the following definitions for the different types of schemas in our databases:

### System Schemas

These are schemas in the database that are completely outside of our control, used by the database itself as necessary. Even though we don't create or manage these, we are including them here for completness and categorization.

### DBA User Schemas

DBA User Schemas will exist for the DBAs to perform necessary functions in our databases.

These schemas will have the highest level of access in our systems, and thus need to be the most careful about credentials and access.

DBA User Schemas should follow this naming convention:

```
{}_DBA
```

Where {} is some useful identifier (i.e. `EXAMPLE_DBA`).

Usually, we have only one of these per database, `EXAMPLE_DBA`. If more are necessary, they should follow this convention.

### Application User Schemas

Application user schemas will exist for each application that needs to access data in our databases.

Application users should be granted the appropriate roles to access the necessary data. They should not be granted individual object grants.

Application users should not have any object creation permissions (i.e. they should not be able to perform DDL operations).

Application User Schemas should follow this naming convention:

```
{}_APP
```

Where {} is the application name (i.e. `CALENDAR_APP`).

### Individual User Schemas

Individual user schemas represent unique individual people that need access to database systems. 

These types of schemas are primarly used by developers and people in the organization that perform data analytics functions.

Individual users should be granted the appropriate roles to access the necessary data. They should not be granted individual object grants.

Individual users should have object creation permissions only for their own user schema.

Individual User Schemas should be named the same as the user's email address. We prefer this to the AD account credentials because it follows a human-readable format, and better allows us to understand who the user is.

### Domain Schemas

Domain schemas are where the actual objects in the database exist (i.e. tables, views, functions, packages).

Domain schemas should have object creation permissions only for their own schema.

When domain schemas are created, the appropriate roles should also be created to be able to control access to the objects.

Note that these might exist in multiple database instances, e.g. custorder might have extract logic in POSP and tables in DSSP.

Domain Schemas should be named logically so their purpose can at least be guessed at.

### Replicated Schemas

Replicated Schmeas are similar to Domain schemas, except they will *only* include data whose system of record is elsewhere. They should not include any objects that aren't represented in the "parent" schema.

Like Domain schemas, the appropriate roles should also be created to be able to control access to the objects.

Replicated Schemas should follow this naming convention:

```
{}_REPL
```

Where {} is the application name (i.e. `CALENDAR_REPL`).

## Consequences

Many of our current systems do not follow these conventions, and we acknowledge that. We may decide to leave large "legacy" schemas (such as the `FOO` schema, or the `BAR` schema) untouched.

## Related Categories

* Database
