# GitHub Actions, Postgres duplication and Docker images

I'm working on a sort of CI/CD pipeline that enables me to focus my efforts on the Java application itself and not the process of testing and deploying. ChatGPT suggested that using GitHub Actions would get me results easier than using Jenkins, and as I want my code to be available in a repository it seemed reasonable to try GitHub Actions first.

## What the pipeline should look like

I want to be able to do push an updated version of the app to GitHub using git via the command line and then have the following things being done automatically:

- The updated code is copied to a Microsoft Ubuntu server where it is packaged and tested
- If the tests are successful, the packaged JAR file is copied to my production server
- On my server a Docker image containing this JAR file and a some version of the JDK is created
- The Docker container that runs the previous version of the JAR file is stopped and removed
- A new Docker container, based on the new image, is started

## Dealing with the database

A common question is how to deal with database access in this process. There are multiple ways, what I do is creating a container with a test database on my production server whose port is exposed to the outside world. It requires a firewall rule that allows access on the exposed port and it allows the settings of the Postgres database to allow access from IP addresses other than that of localhost.

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
	-v pgdatamytestdatabase:/var/lib/postgresql/data 
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

## The GitHub Actions workflow file

GitHub Actions has a simple declarative language in which you write so called 'workflows'. The format is .yml and if you store them in `.github/workflows` in the project root directory they are found by GitHub. I created an extra folder '.github/workflows/unused workflows to store .yml files that I want to keep but not being used.

### The trigger

In your .yml file you start with a name of the workflow followed by the section that describes the trigger that will activate the workflow. Documentation is [here](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow), personally I want my workflow to execute when I push updated code to the main branch of the repository, and I wanted some possibility to execute the workflow manually. This is the code required for that:

```
name: Build, Test and Deploy

on:
  push:		# Executes when code is pushed to main
    branches:
      - main
  workflow_dispatch:  # Provides a button in GitHub that executes the workflow
```

### Build and test

Every 'job' section implies the use of its own virtual machine, or 'runner'. So if you define multiple jobs, they will run parallel on different virtual machines, which is something you might want. In the jobs section you can create sub-sections, each describing a main set of steps. The first sub-section for me is 'build-and-test' and its code looks like this, you find an example from the official documentation [here](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven):

```
jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-java@v5
        with:
          distribution: 'temurin'
          java-version: '21'
          cache: 'maven'

      - name: Build and test
        run: mvn -B verify --file pom.xml

      - name: Package app (skip tests)
        run: mvn -B package -DskipTests --file pom.xml

      - name: Upload JAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar
```

The 'runs-on' field tells on which server type the steps that are defined must run. I will probably always choose 'ubuntu-latest' but Mac and Windows can be chosen as well. The first step, uses: actions/checkout@v5 is something predefined and standard. [Documentation is here](https://github.com/actions/checkout). What it does is copying the repository files to the ubuntu server. 'uses' is used in combination with pre-defined, reusable actions. You can create actions yourself and share them or use those of others, a bit like Docker images.

The actions/setup-java@v5 action sets up a Java JDK, you can specify which one. See [documentation](https://github.com/actions/setup-java). The `cache: 'maven'` setting prevents multiple downloads of the same dependency, see [docs](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven#caching-dependencies). The `run: mvn -B verify --file pom.xml` runs Maven and does verification, which includes testing, and the next step `run: mvn -B package -DskipTests --file pom.xml` does the packaging. Both steps have a name tag which is helpful when reading the log of the proces.

The `actions/upload-artifact@v4` is meant to get the JAR file that was generated from the runner (which is ephemeral) to a place from where you can download it to use it in a next phase of the workflow. I do not know where exactly it is stored but we'll see in the next part that anything uploaded under a certain name can be downloaded under the same name. Note that the 'app-jar' value under the name field is not the name of the file that is uploaded.

### Deploy

The yml file proceeds with a 'deploy' section that starts like this:

```
  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: success()        # only runs if tests passed
```

This code tells GitHub Actions that deploy can only start when the build-and-test phase was successful.

Step one of the deployment phase looks like this:

```
    steps:
      - name: Download built artifact
        uses: actions/download-artifact@v4
        with:
          name: app-jar
          path: app/
```

As you see the download action uses the same name (app-jar) as the upload action. The file is downloaded to `app/*.jar`, the wildcard being its filename. Don't get misled by thinking that app-jar is some file- or directory name, it is not. It is the label given to the totality of artifacts. Anyhow, this whole download action is required because we need to upload this file to our production server. This is done in the next step:

```
      - name: Copy app and build Docker image on server
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          source: "app/*.jar"
          target: "~/deploy/"
```

Apparently appleboy created some copy action that allows us to copy  the jar file to our server in an efficiënt way. As aprguments it uses so-called secrets, you can set them somewhere in the GitHub settings to prevent visitors of your repository from stealing them. It is remarkable that here the .jar extension is used again, while it was omitted in the previous step. The jar file is copied to `/home/geertjan/deploy/app/*.jar` on the production server. The name of the jar file, I think, is the name as created by Maven during packaging. This name is never mentioned in this yml file because it can't be known.

The step below does the final thing, namely connecting to the production server (like in the previous step) and build a new Docker image based on the last version of the jar file that has just been uploaded. This is what the build command does and I suspect that this can only work when the appropriate docker file is in the same folder on the production server as the jar file will land in (/home/geertjan/deploy)

```
      - name: SSH and deploy Docker container
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          script: |
            cd ~/deploy
            docker build -t myapp:latest .
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp -p 8080:8080 myapp:latest
```

I have not run this last step for real (all previous steps worked) so I need to still figure out what the Dockerfile must look like. The one I have on my work pc is this, from ChatGPT:

```
FROM amazoncorretto:21
WORKDIR /app
COPY target/mijnwoonplaats-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

I suspect that I can do without the COPY line but as the jar file already resides in /home/geertjan/deploy/app the WORKDIR directive should stay in place. I do not know if I can use wildcards in Dockerfiles, because that is what I would need to get rid of the long-name-that-I-do-not-know and replace it by app.jar.




















