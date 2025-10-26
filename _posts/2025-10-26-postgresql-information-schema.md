# PostgreSQL Information Schema

It has been a while since my last post but have been working hard on getting better at Docker, Linux, GitHub Actions and PostgreSQL. Of the latter I'm seriously trying how much of the (quite superb) documentation I can stuff in my head. 

Understanding the internals of Postgres means a lot of things, one of them being the way that Postgres stores many of the internals in the database itself, more specifically in the tables of schema pg_catalog. If you create objects like functions or roles, or if you grant permissions to roles, or if you alter the default persmissions of roles, all this information is stored in schema pg_catalog. 

To make this somewhat obscure information accessible there are the views in the pg_catalog, and there are the views in schema information_schema. The latter applies a filter on the data based on the permissions of the current user. If you check out these views as superuser, you can piece together the puzzle and find out who has what permissions on what.

The most convenient way to find out such things is with using pasql commands like `\z`, `\ddp`, but there might be situations where you want to write your own queries/functions that provide more precise insight into internals. Tip: to see the queries behind psql '\' style commands, use `\set ECHO_HIDDEN on`. Once you use the command, the query is printed to the shell which allows you to see how it is formulated. `\set ECHO_HIDDEN off` to revert to normal.

## The Information Schema

In the current version (18) there are 64 views in information_schema. Below is a table in which I give a description and a 'relevance' flag for each.

|Relevance|View name|Description|
|----|----|----|
||information_schema_catalog_name|Contains only name of current database.|
|\u2705|administrable_role_​authorizations|Identifies all roles that the current user has the admin option for.|
|\u2705|applicable_roles|Identifies all roles whose privileges the current user can use.|
||attributes|Information about the attributes of composite data types defined in the database.|
||character_sets|Character sets available in the current database. Since PostgreSQL does not support multiple character sets within one database, this view only shows one, which is the database encoding.|
||check_constraint_routine_usage|Routines (functions and procedures) that are used by a check constraint. Only those routines are shown that are owned by a currently enabled role.|
||check_constraints|All check constraints, either defined on a table or on a domain, that are owned by a currently enabled role. The owner of the table or domain is the owner of the constraint.|
||collations|Collations available in the current database. Collation is about sort order and case conversion, which can be language specific.|
||collation_character_set_​applicability|Identifies which character set the available collations are applicable to.|
||column_column_usage|All generated columns that depend on another base column in the same table. Only tables owned by a currently enabled role are included.|
||column_domain_usage|All columns (of a table or a view) that make use of some domain defined in the current database and owned by a currently enabled role.|
||column_options|All the options defined for foreign table columns in the current database. Only those foreign table columns are shown that the current user has access to.|

\u2705




