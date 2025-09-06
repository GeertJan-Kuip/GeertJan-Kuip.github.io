# SSH and Linux

As I need to deploy my application on a server I need to be more comfortable with running a Linux system via the command line. I started to work on it and this is a summary of what I have done and learnt. More to follow.

## WSL and Ubuntu

On Windows 11, more exactly the newer versions, you can run ```wsl --install``` which enables WSL 2. WSL stands for Windows Subsystem for Linux. WSL is not a full virtual machine but it allows you to run Linux applications alongside your Windows apps.

Enabling WSL 2 is not enough to do Linux things, you need to install a so-called Linux 'distro'. Ubuntu is the most popular, Debian is more minimalistic, and there are other options as well with specific strengths. You can install multiple distros and switch between them.

## GUI

It used to be that WSL was command line only but now you can run Linux applications with real GUI's. Think of Firefox, Intellij or VS Code Linux edition.

Apart from that there are also all sorts of Linux command line tools like bash, ssh, vim, python or git that you can run.

## Running external server

I bought a Hetzner CX22 server for about 5 euro a month and need to direct it. Practically I need to run Java programs and, for now, at least one PostgrSQL database, either in Dockers or just straight on the Linux OS. The server has a dashboard but not much in the form of preinstalled tooling, so it must be done via the command line.

### SSH Key

One thing during setup was the question whether I wanted an SSH key, according to ChatGPT that was a good idea so I generated one in my Ubuntu terminal. The code to generate one is:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

The mail address is the one you signed up with. You will be prompted for the file pathe (where to store the keys, use the default) and you'll be asked if you want to add a passphrase. If yo choose a passphrase, you will need to enter that on access, if you skip it, logging in to your server goes without you having to supply any password or code.

The key has a private and a public variant, you will share the public key with, in this case, Hetzner. To get it fromout the Ubuntu terminal type:

```
cat ~/.ssh/id_ed25519.pub
```

Get the key, including a prefix at the begin and your email address at the end, and enter it in the form field of the Hetzner website. 

If you have generated the ssh keys in the Ubuntu terminal, they are stored in `~/.ssh/`. This is a folder that you will not be able to find browsing in your Windows explorer, it is hidden somewhere in the virtual Linux environment. You can access this folder in your Windows file explorer by typing '\\wsl$\Ubuntu\home\geert\.ssh\' in the path field. Note that 'geert' can be something else, depending on how user names are set.

The folder contains two keys, the one ending on .pub is the public key. The other one should never be shared as it is the private key.

## Access and update server

From the Ubuntu terminal, run:

```
ssh root@<ip address of server>
```

The ip address is to be found on the online dashboard of Hetzner. I was asked in ordering the server whether I wanted only free IPv6 addresses or also IPv4, ChatGPT advised to have both. I use the IPv4 address to log in. 

On first access, ChatGPT advised to update the server with `sudo apt update`. It worked perfectly.

## Install Java

To install Java, I used 'sudo apt install openjdk-21-jdk -y`. I had created my application for Java 21 and had some trouble creating the uberjar with Maven as my environment settings had Java 17 as first one in the order of paths, it took me some time to figure it out. It was confusing as in Intellij Java 21 was the one that ran. Downgrading was problematic because the code included something with enums and possibly a switch statement that was valid in 21 but not in 17. Anyway, it is good to have Java in just one version for now.

Btw if you run your application in a Docker container, you can choose any version of Jave when you select the image. So if I will add dockerized apps in the future I'm not bound to the 21 version.

## Install PostgreSQL

This required a bit more instructions because environment variables had to be set for database access. I choose to add environment variables DB_URL, DB_USER AND DB_PASS to both my own machine and to the server and refer to them in application.properties. For my own windows pc I did it via the 'Omgevingsvariabelen' menu, although it can be done directly via Powershell, on the server it required something more exotic.

The first step to install PostgreSQL on the server is by this line:

```
apt install postgresql postgresql-contrib -y
```

I don't know what the postgresql-contrib exactly stands for but this line is probably typed 1000 times per hour worldwide. After installation, status can be checked by ```systemctl status postgresql```. You de q to exit.

## Create database with user

I'm not sure why PostgrSQL could or should be installed without administrator permissions, without sudo. Anyhow, this is what ChatGPT told me to do:

Switch to the postgres user:

```
sudo -i -u postgres
```

Open psql:

```
psql
```

Inside psql, create db and user. Following lilnes need to be entered one by one:

```
CREATE DATABASE mydatabase;
CREATE USER myuser WITH PASSWORD 'astrongpassword';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO myuser;
\q
exit
```

## Copying the JAR to the server

To copy the fat jar to the server, type this:

```
scp target/myapp-0.0.1-SNAPSHOT.jar root@<server ip>:/root/
```

Okay, I need to do this, and learn how to navigate the file su=ystem in Linux. To be continued.





