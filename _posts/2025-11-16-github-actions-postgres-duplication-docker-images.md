# GitHub Actions and Docker images

I'm working on a sort of CI/CD pipeline that enables me to focus my efforts on the Java application itself and not the process of testing and deploying. ChatGPT suggested that using GitHub Actions would get me results easier than using Jenkins, and as my code needs to available in a repository it seemed reasonable to try GitHub Actions first.

## What the pipeline should look like

I want to be able to do push an updated version of the app to GitHub using the Git command line and then have the following automated process:

- The updated code is copied to a Microsoft Ubuntu server where it is packaged and tested
- If the tests are successful, the packaged JAR file is copied to my production server
- On my server a Docker image containing the JAR file and a some version of the JDK is created
- The Docker container that runs the previous version is stopped end removed
- A new Docker container, based on the new image, is started

## Dealing with the database

A common question is how to deal with database access. There are multiple ways, what I do is to create a container with a test database on my production server whose port is exposed to the outside world. It requires a firewall rule that allows access on the exposed port and it allows the settings of the Postgres database to allow access from IP addresses other than that of localhost.

### Copying the database with pg_dump

To create a copy of a dockerized PostgreSQL database you can use the [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html) command as part of the docker exec function:

```
sudo docker exec {name_of_postgres_container} pg_dump {name_of_database} -O -U {username} > mydumpfile.sql
```

This command executes the pg_dump command inside the container and writes the output to "mydumpfile.sql", which will be created in the current directory (not in the container filesystem).  The -O flag (--no-owner) results in discarding all sql statements related to ownership. 

### Create a new container with a test db

To create a new Docker container which runs as the test database based on the copy and is accessible from the outside, run this command:

```
sudo docker run --name {mycontainerfortestdb} 
	--restart=unless-stopped 
	-e POSTGRES_DB={mytestdatabase} 
	-e POSTGRES_USER={someusername} 
	-e POSTGRES_PASSWORD={somepassword}  
	-v pgdatamijnwoonplaatsdbtest:/var/lib/postgresql/data 
	-p 5555:5432 
	-d postgres:16
```

The `--restart=unless-stopped` guarantees that the container will be restarted upon server restart. The -v flag guarantees the data persists, no matter the restarts. The -p flag sets the port on which the container, and therefore the database, can be accessed. It cannot be 5432 because that port is probably used by the production db. ChatGPT suggested using some obscure number, which at least will mislead the most basic server sniffers.

### Insert the data of the dump file

The new container has a running database in it but there are no tables and rows inside it. We need to copy the 'mydumpfile' into it. To do so, we need to use psql, either from an external machine or from the server itself :

```
psql -h {server_ip_address} -p 5555 -U {someusername} -d {mytestdatabase} #external access

psql -h localhost -p 5555 -U {someusername} -d {mytestdatabase} #internal access on server itself
```

The latter of the two options above is the easiest as you will need access to the sql dump file. The command to load the data in the dump file into the database is this:

```
mytestdatabase=# \i mydumpfile.sql
```

The description of the `\i` command is: _Reads input from the file filename and executes it as though it had been typed on the keyboard._. It means that it will recreate the database and that is exactly what you want. Btw the tables are created with SQL statements, the insertion of rows is done with a copy function which is much faster.

### Firewall and PostgreSQL listen address

The firewall settings can be adjusted on the console of my server. To give the outside world access, the port used on the container for the test db must allow access. ChatGPT suggested to limit this outer access to the IP range of GitHub Actions (you can google it I think) plus the IP address of your work pc. 

Opening the firewall is not enough, PostgreSQL itself has a relevant setting as well. The setting that opens the Postgres database to outside connections is to be find in postgresql.conf. If the db is in a container you can view this file like this:

```
sudo docker exec mijnwoonplaatscontainertestdb cat /var/lib/postgresql/data/postgresql.conf
```

The file is long but you need to find 'listen address':

```
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'
```

In the Postgres Docker image I used the default setting for listen_address is '*', meaning that calls from the outside are allowed. This means you do not have to adjust it. On a 'bare metal' Postgresql installation the default setting is 'localhost', which means you need to adjust it. It is good to be aware of this.


