# Systemd Units

When asking ChatGPT about how to run the Linux server it proposed to write a .service file and store it in `/etc/systemd/system`. I had no clue how this worked but figured it out today. Got some proper explanation from the [Learn Linux TV](https://www.youtube.com/watch?v=Kzpm-rGAXos) YouTube channel and from a [Digital Ocean](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) post.

## Systemd

Systemd is the default initialization system for Linux. It is the manager that on startup calls all sorts of processes called units. You can configure how and when, or under what conditions, these processes are called and you do this by editing the unit files.

### Systemd and the systemctl command

Services can be managed via the command line via the `systemctl` command. You can start, stop and restart a service, and you can enable or disable it. The latter specifies whether the service will automatically start at startup. These options give you all possible control but do not get you anywhere if it is about elementary services that need to run to make Linux work. Starting up Linux requires many services to start up in a carefully orchestrated manner. That is where the unit files come in.

### Unit files

A unit file is a short and simple text file, written in a simple declarative style, that tells systemd how, when and under what conditions to do what. The most common unit file is of .service type and provides a command that needs to be executed. Other types of unit files, like .timer or .path, do not in itself provide a command to run but form a combination with some .service file. A .path file for example tells systemd to monitor a specific location and to start some service when specific events are detected. A .timer file specifies the time(s) upon which systemd must call a specific service.

I [found](https://docs.oracle.com/en/operating-systems/oracle-linux/8/systemd/SystemdUnits.html) this list of available unit types:

- service
- mount
- swap
- socket
- target
- device
- automount
- timer
- path
- slice
- scope

You recognize the unit file type on its postfix, which correspons to these terms.

### Where the unit files live

There are three locations. Files with identical names override each other whereby the precedence order (low to high) is:

- /lib/systemd/system
- /run/systemd/system
- /etc/systemd/system

Linux directories all serve a specific purpose, `lib` is for libraries needed to boot the system and run the Linux commands, `run` has a temporary character and for example contains the process identifier (PID) files, while `etc` is the directory where all sorts of user-manageable configuration lives. Groups, users, passwords can be found here. If you create your own unit file, this is the place to store it.

### Customizing/overriding unit files

Because files in `etc` override those in `lib`, you do not have to access the vulnerable `lib` folder to make adjustments to the unit files that live there. Instead, you override them with a similarly named file in `etc`. If for some reason you want to revert to the default, just delete the override file.

Linux has some built-in functionalities (like `systemctl edit`) to create override files in etc, instructions can be found [here](https://youtu.be/Kzpm-rGAXos?si=gjz2nnVjVaYTsT8b&t=1982).

### Anatomy of a unit file

You can read examples of unit files by listing the contents of `/lib/systemd/system` and use `cat` on any one of the files you find here. They mostly consist of three sections that start with some case-sensitive header between brackets. Systemd recognizes these section headers. Subsequently you find key-value pairs. This [article](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) lists all sorts of keys that can be used.

#### [Unit]

Most unit files start with this section. Here you find metadata (Description and Documentation) and all sorts of values that indicate how the unit relates to other unit. Think of dependencies, conflicts to be avoided or spedific conditions that must be met. This is important as systemd must know if the unit must be run and if so, what the order will be in which they will be run.

#### Unit-Specific Section Directives (like [Service],[Timer],[Mount] etc)

The name of this section is identical to the type but starts with a capital. As most unit files are of service type, this is the one you will find most. There is a whole list of possible keys you can use, among them a list of directives starting with 'Exec'. These specify the command that must be run. [The article](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) has a listing.

Note that in non-service unit files there is often no 'Exec' directive. When it is missing, by default the thing to execute is the unit file that has the same name but the .service extension. In the `/lib/systemd/system` folder for example there is a `apport-autoreport.timer` and a `apport-autoreport.path` unit file that both do not have some Exec directive. But there is also a `apport-autoreport.service` unit file that does have an Exec directive. Timer and path implicitly tell systemd to call their .service counterpart, and here systmed reads what command to execute.

#### [Install]

This section is optional. Only units that can be enabled/disabled have this section.

## My case

I need a simple .service unit file that has some Exec directive in the [Service] section. The value must be something like `java -jar mijnwoonplaats.jar`. This unit file must tell systemd that it should start my Spring application upon startup and also upon a crash of my application. Possibly in the Unit section it must be told that my application can only be started after PostgreSQL, given its dependence on the database.

### What ChatGPT proposed:

This is what ChatGPT came up with, before I even knew what a service or a systemd unit was:

```
# /etc/systemd/system/mijnwoonplaats.service
[Unit]
Description=Mijn Woonplaats Spring Boot App
After=network.target

[Service]
User=root
ExecStart=/usr/bin/java -jar /root/mijnwoonplaats-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

I wonder if the PostgreSQL dependency must have a place in it. Furthermore there will be some changes in the user, probably root will be disabled for safety. And there must be some way that the filename of myapp is not hard-coded, as I shouldn't have to change this file every time the version changes. 

We'll talk about it.


 
