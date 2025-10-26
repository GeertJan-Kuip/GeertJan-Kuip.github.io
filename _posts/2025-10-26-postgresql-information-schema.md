# PostgreSQL Information Schema

It has been a while since my last post but have been working hard on getting better at Docker, Linux, GitHub Actions and PostgreSQL. Of the latter I'm seriously trying how much of the (quite superb) documentation I can stuff in my head. 

Understanding the internals of Postgres means a lot of things, one of them being the way that Postgres stores many of the internals in the database itself, more specifically in the tables of schema pg_catalog. If you create objects like functions or roles, or if you grant permissions to roles, or if you alter the default persmissions of roles, all this information is stored in schema pg_catalog. 

To make this somewhat obscure information accessible there are the views in the pg_catalog, and there are the views in schema information_schema. The latter applies a filter on the data based on the permissions of the current user. If you check out these views as superuser, you can piece together the puzzle and find out who has what permissions on what.

The most convenient way to find out such things is with using pasql commands like `\z`, `\ddp`, but there might be situations where you want to write your own queries/functions that provide more precise insight into internals. Tip: to see the queries behind psql '\' style commands, use `\set ECHO_HIDDEN on`. Once you use the command, the query is printed to the shell which allows you to see how it is formulated. `\set ECHO_HIDDEN off` to revert to normal.

## The Information Schema

In the current version (18) there are 64 views in information_schema. Below is a table in which I give a description and a 'relevance' flag for each.

\u2705

:white_check_mark:


