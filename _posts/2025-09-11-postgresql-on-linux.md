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
- `postgres=#` is what you see. Here postgres is the 'superuser' that can play god. The '#' tells you that the active user is a superuser (postgres can make other users superuser as well).
- You can change the active user with `SET ROLE someotherusername;` Always use `SET ROLE ...;` 
- `\q` and `exit` will get u out of things


### Cluster, database(s) and schema('s)

A PostgreSQL db has a structure that begins with a cluster, which you won't notice, and below it one or more db's can be created. Within the individual databases there are [schema's](https://neon.com/postgresql/postgresql-administration/postgresql-schema), which act as namespaces and help to keep things orderly. There is one default schema called 'public' and if you do not create additional schema's you will hardly notice its existence.

- `\l` shows the databases. You can see who the owner is, that is primarily the role that created the db
- `\c {databasename}` bring you into the specified database
- `\d` shows tables in db. For compact formatting try `\a`, for omitting column titles `\t`.
- `\d {tablename}` shows only the specified table
- `TABLE {tablename} LIMIT {n};` shows tablecontent, first n rows. Don't forget semicolon as this is real sql

### Roles

PostgreSQL uses the concept of roles, although in its earlier days it used the concepts of users and groups. A role can be a user or a group, a group is a role with the NOLOGIN attribute. Groups function as parents of which privileges can be inherited which is convenient for administration.

When creating a role you can set multiple attributes that define the power of the role. Typical attributes are SUPERUSER, CREATEDB, CREATEROLE, LOGIN and PASSWORD. See [here](https://www.prisma.io/dataguide/postgresql/authentication-and-authorization/role-management) for explanation. 


