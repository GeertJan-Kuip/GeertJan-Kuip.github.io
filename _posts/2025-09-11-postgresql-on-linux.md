# PostgreSQL on Linux

I installed PostgreSQL on my Ubuntu Linux server and stumbled upon some problems having to do with access and permissions. I decided to figure things out and in this blog post I'll try to describe not how you get things done, but how you can inspect the state of the PostgreSQL server, its contents and the status and privileges of its users.

## Checking if PostgreSQL runs

Run this in the Ubuntu shell as root or other user.

- `systemctl status` gives list of systemd units. There should be postgres stuff in it.
- `systemctl status postgresql` provides the following if postgres runs well:

```
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
     Active: active (exited) since Sat 2025-09-06 19:18:55 UTC; 5 days ago
   Main PID: 894 (code=exited, status=0/SUCCESS)
        CPU: 4ms
```

The first 'enabled' tells you it works, the second that postgres will be started automatically on server reboot. The 'active' word means postgres is running fine.

## Into the database

### Commands

- Enter `su postgres` to switch from root- or other user to the user named postgres. This Linux user was created upon postgres installation.
- Enter `psql` to start the psql program that lets you talk more directly to the db. 
- `postgres=#` is what you see. Here postgres is the 'superuser' that can play god. The `#` tells you that the active user is a superuser (postgres can make other users superuser as well). If no superuser, you see `>` instead of `#`.
- You can change the active user with `SET ROLE someotherusername;`Use `SET ROLE postgres;` to revert. 
- `\q` and `exit` will get u out of things


### Cluster, database(s) and schema('s)

A PostgreSQL db has a structure that begins with a cluster, which you won't notice, and below it one or more db's can be created. Within the individual databases there are [schema's](https://neon.com/postgresql/postgresql-administration/postgresql-schema), which act as namespaces and help to keep things orderly. There is one default schema called 'public' and if you do not create additional schema's you will hardly notice its existence.

- `\l` shows the databases. You can see who the owner is, that is primarily the role that created the db
- `\c {databasename}` bring you into the specified database
- `\dn` lists the schema's and their owner
- `\d` shows tables in db. For compact formatting try `\a`, for omitting column titles `\t`.
- `\d {tablename}` shows only the specified table
- `TABLE {tablename} LIMIT {n};` shows tablecontent, first n rows. Don't forget semicolon as this is real sql

### Roles and attributes

PostgreSQL uses the concept of roles, although in its earlier days it used the concepts of users and groups. A role can be a user or a group, a group is a role with the NOLOGIN attribute. Groups function as parents of which privileges can be inherited which is convenient for administration.

When creating a role you can set multiple attributes that define the power of the role. Typical attributes are SUPERUSER, CREATEDB, CREATEROLE, LOGIN and PASSWORD. See [here](https://www.prisma.io/dataguide/postgresql/authentication-and-authorization/role-management) for explanation. The typical statement can look like:

```
CREATE ROLE geert WITH LOGIN PASSWORD 'hehe' CREATEDB;
```

This created a user that can login (and is thus not a group), has a password 'hehe' and has the power of creating databases, which is often not a good idea.

Commands to know:

- `\dg` or `\du` lists all roles + attributes (you might say users but roles is the correct term)
- `SELECT SESSION_USER, CURRENT_USER;` lists the default user in sessions (often postgres) and the current user (via `SET ROLE ..;`)
- `SET ROLE {username};` changes current user. Use it if you want another user than default to own created objects.

Good link [here](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps-2).

### Ownership and privileges

The user that creates a database, schema or table (or any other object, there are more) becomes the owner. Only the owner can drop or alter the object. Ownership can be transferred to another user with statement like `ALTER SCHEMA public OWNER TO someother;`. 

Other privileges, think of writing to tables or creating new tables in a database you do not own, can be granted to by owner or the superuser. Without this grant a non-privileged user cannot do anything. The owner of an object on the other hand has all privileges on that specific object.

The confusing thing is that to become an owner, you must create the object, and to create the object, you must first have been granted the right to do so by either the superuser or the owner of the object one level above the object you create yourselves. Examples:

- Owner of database can grant the privilege to add a schema to selected user(s)
- Owner of schema can grant the privilege to add a table to selected user(s)
- Owner of table can grant the privilege to add a row to the table to selected user(s)
- Owner of database can NOT grant the privilege to add a table to selected user(s), unless he also owns the schema
- Owner of schema can NOT grant the privilege to add a row to a table to selected user(s), unless he also owns that table

Typical GRANT statement by owner of schema in which mijntabel lives (btw capitals are not mandatory):

```
grant all privileges on table mijntabel to henk;
```

More syntax [here](https://neon.com/postgresql/postgresql-administration/postgresql-grant).

### Default privileges

You can set default privileges for roles with a statement starting with `ALTER DEFAULT PRIVILEGES`. This translates in new database content in the `pg_default_acl` table, which is empty by default. 

It seems to be important that you set the default privileges before creating the objects for which you want to have them, it cannot be fixed retrospectively. 

This [link](https://www.postgresql.org/docs/current/sql-alterdefaultprivileges.html) from the official docs explains a bit.

### Difference between attributes and privileges

The right to create a database or a role is not a privilege but an attribute. it is assigned on creation of the role with a statement like this:

```
CREATE ROLE geert WITH LOGIN PASSWORD 'hehe' CREATEDB;
```
It can also be assigned after initial creation of the role using an ALTER statement:

```
ALTER ROLE demo_role WITH LOGIN;
```

While the power to create a database or a role is just an attribute carried by a user (I should say role), a privilege is very specifically tied to objects and thus can not be generalized to an attribute. You do not have the privilege to add tables, you only have the privilege to create them in a specific schema. In the listing of roles (`\du`  or `\dg`) you can see the attributes easily but this is harder to see for all the different objects. 

### Querying custom privileges

PostgreSQL stores privileges in the so-called system catalogs, which are special tables in the schema `pg_catalog`. As privileges, unlike role attributes, are manifold and complex, there are no simple commands that will tell you all. Nevertheless ChatGPT provided a query that gave me a list of all the specific privileges I had granted to this user named geert.

```
SELECT grantee, privilege_type, table_schema, table_name
FROM information_schema.role_table_grants
WHERE grantee = 'geert';
```

I have not figured out how in what structure the privileges are exactly stored but this hack for now is enough.

### Working with local Dockerized PostgreSQL db

All of the above deals with undockerized PostgreSQL databases. I have a dockerized db locally and wondered how to use psql on it. This is the way to get access to it:

```
psql -h localhost -p 5432 -U <usernamedb> -d <nameofdatabase>
```

You are prompted for the password. The connection you get is the same connection as your application has, and this is (naturally) sufficient to use psql commands. You do not need the name of the container, knowing the port that is exposed by the container is enough.

Check port number, 5432 is default but in case you might have created the Docker image with another port mapped, you should use that.

You can also work via `docker execute`, it means you approach the database from inside the container:

```
docker exec -it <container_name_or_id> psql -U <username> -d <database>
```

In this case you need the container name or id, you find it in Docker desktop with an easy copy option. The command brings you to the typical psql mode with `db_mijnwoonplaats=#`. I noted that the db in my container had only one role, which was not postgres but the username/role I created upon creating the container, and that it is a superuser. The `#` confirmed this superuser status.


