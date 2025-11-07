# Bash scripting part 2

In a previous post I wrote about bash scripting and gave a specific example of a script meant to create a command that would provide all sorts of info about the Docker installation on a Linux server, including the systemctl status of the daemon and the individual containers, their connection to networks, the images on which they are based, their internal and external ports, the duration of them running, the command executed in them, the restartpolicy and the mounts inside. 

I created other scripts as well and combined them all so that with one command, they all run. This results in extensive terminal output that tells about the following:

- Users and permissions
- The contents of the home directory, including directory tree
- Contents of /etc/systemd/system
- Everything about TCP ports and their connected processes
- Info on Nginx
- Docker (described in previous blog)

## How I organized it

In my home folder /home/geert I created a scripts folder.

```
└── scripts
    ├── dockerinfo.sh
    ├── etc-systemd-system.sh
    ├── homedir.sh
    ├── main.sh
    ├── nginx.sh
    ├── ports.sh
    └── userinfo.sh
```

The main.sh script is the collection point for all the other scripts. It looks like this:

```
bash ~/scripts/userinfo.sh
bash ~/scripts/homedir.sh
bash ~/scripts/etc-systemd-system.sh
bash ~/scripts/ports.sh
bash ~/scripts/nginx.sh
bash ~/scripts/dockerinfo.sh
```

This means that running the main script runs everything. The alias I created for main.sh is 'main', and to make it work across sessions I stored the alias in a file `~/bash_aliases`. This file is loaded by `~/bashrc`, which is the place to go for bash related configuration. In `~/bash_aliases` you only store alias commands. Mine looks like this (tellme is another command I defined, the file is elsewhere):

```
alias tellme='bash ~/tellme.sh'
alias main='bash ~/scripts/main.sh'
```

In ~/.bashrc you find the code that loads the aliases, this is the code:

```
# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

## Home directory

I created a script that gathers info on the home directory of the active user. Its relevant code is just this (I omitted layout related commands):

```
ls -a  # lists all files and folders in a compact way
echo ""

tree ~ --filesfirst  # Shows the home folder contents as file/directory tree
```

The tree command is not Linux native but must be installed with `sudo apt install tree -y`. Documentation is [here](https://linux.die.net/man/1/tree). It provides excellent insight into the deep contents of a folder, something that is otherwise hard in cli Linux. The total output in this section looks like this:

```
.  ..  dockerinfo.sh  etc-systemd-system.sh  homedir.sh  main.sh  nginx.sh  ports.sh  userinfo.sh

/home/geertjan
├── dump01.sql
├── func.sh
├── kort.sh
├── mijnwoonplaats-0.0.1-SNAPSHOT.jar
├── myscript.sh
├── tellme.sh
├── tellpg.sh
├── deploy
│   └── app
│       └── mijnwoonplaats-0.0.1-SNAPSHOT.jar
└── scripts
    ├── dockerinfo.sh
    ├── etc-systemd-system.sh
    ├── homedir.sh
    ├── main.sh
    ├── nginx.sh
    ├── ports.sh
    └── userinfo.sh

4 directories, 15 files
```

## /etc/systemd/system

I created a section in the output that contains all files in folder /etc/systemd/system. The reason is that this is the logical location for any custom unit files you might want to add. If you see all its contents you can immediately spot the ones you added yourself. 

Apart from layout-related stuff, the script contains only this line:

```
ls /etc/systemd/system -a
```

Output:

```
.                                       emergency.target.wants     ModemManager                    sleep.target.wants                   syslog.service
..                                      final.target.wants         multi-user.target.wants         sockets.target.wants                 sysstat.service.wants
cloud-final.service.wants               getty.target.wants         network-online.target.wants     sshd-keygen@.service.d               timers.target.wants
cloud-init.target.wants                 hibernate.target.wants     oem-config.service.wants        ssh.service.requires                 vmtoolsd.service
dbus-org.freedesktop.resolve1.service   hybrid-sleep.target.wants  open-vm-tools.service.requires  suspend.target.wants
dbus-org.freedesktop.timesync1.service  iscsi.service              paths.target.wants              suspend-then-hibernate.target.wants
display-manager.service.wants           mdmonitor.service.wants    rescue.target.wants             sysinit.target.wants
```

## TCP ports and processes

You want to know what ports are listening, established or waiting and what processes are connected to it. It provides insight in what applications are active and eventually which outside users are connected or trying to connect.

The relevant bash code I created is this: 

```
echo "Command: sudo ss -antp. Note: on WSL2 Ubuntu this list will be incomplete."
echo ""
sudo ss -antp
```

It is very little but great for insight. Note that it requires sudo, which might mean you will be prompted for a password during the execution of this script which is no problem. 

The `sudo ss -antp` worked well for me, I learned that `ss` is a better successor of `netstat` so I use it. The flag -a means that both listening and non-listening sockets must be displayed, -t limits the output to TCP ports (otherwise you'll get a ton of internal Unix ports) and -p displays the process name. The -n description says 'Do not try to resolve service names.' I just noticed that you can omit this flag and have the same result. Anyway, extensive info available via the man page or man command.

Output:

```
Command: sudo ss -antp. Note: on WSL2 Ubuntu this list will be incomplete.

[sudo] password for geertjan:
State       Recv-Q       Send-Q             Local Address:Port               Peer Address:Port       Process
LISTEN      0            511                      0.0.0.0:80                      0.0.0.0:*           users:(("nginx",pid=6842,fd=5),("nginx",pid=6841,fd=5),("nginx",pid=6840,fd=5))
LISTEN      0            4096                     0.0.0.0:22                      0.0.0.0:*           users:(("sshd",pid=1411,fd=3),("systemd",pid=1,fd=132))
LISTEN      0            4096               127.0.0.53%lo:53                      0.0.0.0:*           users:(("systemd-resolve",pid=579,fd=15))
LISTEN      0            4096                  127.0.0.54:53                      0.0.0.0:*           users:(("systemd-resolve",pid=579,fd=17))
LISTEN      0            4096                   127.0.0.1:43697                   0.0.0.0:*           users:(("containerd",pid=805,fd=11))
LISTEN      0            4096                     0.0.0.0:8080                    0.0.0.0:*           users:(("docker-proxy",pid=1056025,fd=4))
LISTEN      0            4096                     0.0.0.0:5678                    0.0.0.0:*           users:(("docker-proxy",pid=1056401,fd=4))
ESTAB       0            72                  91.98.68.196:22                86.80.245.106:64194       users:(("sshd",pid=1075345,fd=4),("sshd",pid=1075252,fd=4))
LISTEN      0            511                         [::]:80                         [::]:*           users:(("nginx",pid=6842,fd=6),("nginx",pid=6841,fd=6),("nginx",pid=6840,fd=6))
LISTEN      0            4096                        [::]:22                         [::]:*           users:(("sshd",pid=1411,fd=4),("systemd",pid=1,fd=133))
LISTEN      0            4096                        [::]:8080                       [::]:*           users:(("docker-proxy",pid=1056033,fd=4))
LISTEN      0            4096                        [::]:5678                       [::]:*           users:(("docker-proxy",pid=1056410,fd=4))
```

## Nginx

The code in the nginx.sh file is this:

```
sudo systemctl status nginx | grep -m 3 ''  # First 3 lines of systemctl status, lets you see if its enabled and/or active
echo ""

sudo netstat -plunt | grep nginx  # ports related to nginx process
echo ""

#sudo cat /etc/nginx/nginx.conf | grep -m 3 ''  # Not included but nginx.conf contains the relevant configuration 
#echo ""

sudo nginx -t  # tells you if everything is fine with nginx
echo ""

echo "ls /etc/nginx"
sudo ls /etc/nginx  # contents of nginx configuration directory
```

While I do not yet use Nginx on my server, I have only installed it, here is the output that results from the previous script:

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-10-06 12:26:44 UTC; 1 month 1 day ago

tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      6840/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      6840/nginx: master

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

ls /etc/nginx
conf.d        fastcgi_params  koi-win     modules-available  nginx.conf    scgi_params      sites-enabled  uwsgi_params
fastcgi.conf  koi-utf         mime.types  modules-enabled    proxy_params  sites-available  snippets       win-utf
```

Quite usefull, and later on I will adjust it so that more information about the configuration can be read.

## Docker

I explained the Docker script already in the previous blog on this topic but here it is again. It is more complex, as it iterates over specific output generated by various Docker commands. Furthermore, there is json output to be parsed. It needs to so because it wants to provide in-depth info on Docker networks and containers.

```
sudo systemctl status docker  | grep -m 3 ''
echo ""

echo "Note: output here depends on Docker running and containers being started."
echo ""

sudo docker ps --all
echo ""

QUERY2=$(cat <<'EOF'
SELECT * FROM buurten LIMIT 5;
EOF
)

do_query () {
  docker exec -it mijnwoonplaatsdbcontainer psql -U user01 -d db01 -At -c "$1"
}

#do_query "${QUERY2}"  // I disabled this one but you can do psql db query like this

sudo docker network ls
echo""

for netwrk in $(sudo docker network ls --format '{{.Name}}'); do   # Here is Go/Docker syntax involved, ls --format is not shell syntax here.
  if [[ "$netwrk" != "host" && "$netwrk" != "bridge" && "$netwrk" != "none" ]]; then
    echo -e "Network \e[4;37m$netwrk\e[0m includes the following containers (only shown if running):"
    for name in $(sudo docker network inspect "$netwrk" | jq -r '.[0].Containers | to_entries[] | .value.Name'); do
      echo -e "  \u2022 $name"
    done
  fi
done

echo ""
echo "CONTAINER DETAILS"

for container in $(sudo docker ps --format '{{.Names}}'); do
  echo -e "\u25AA \e[4;37m$container:\e[0m"
  #echo "$container:"
  NETWORKNAME=$(sudo docker inspect "$container" | jq -r '.[0].HostConfig.NetworkMode')
  RESTARTPOLICY=$(sudo docker inspect "$container" | jq -r '.[0].HostConfig.RestartPolicy.Name')
  echo "  Network: ${NETWORKNAME}"
  echo "  RestartPolicy: ${RESTARTPOLICY}"

  MOUNT_TYPE=$(sudo docker inspect "$container" | jq -r '.[0].Mounts.[0].Type')
  MOUNT_NAME=$(sudo docker inspect "$container" | jq -r '.[0].Mounts.[0].Name')
  MOUNT_SOURCE=$(sudo docker inspect "$container" | jq -r '.[0].Mounts.[0].Source')
  MOUNT_DESTINATION=$(sudo docker inspect "$container" | jq -r '.[0].Mounts.[0].Destination')
  echo "  Mounts (only first):"
  echo "    Type: ${MOUNT_TYPE}"
  echo "    Name: ${MOUNT_NAME}"
  echo "    Source: ${MOUNT_SOURCE}"
  echo "    Destination: ${MOUNT_DESTINATION}"
  echo ""
done
```

As you see this script is more challenging. Some syntax explanations:

```
for netwrk in $(sudo docker network ls --format '{{.Name}}'); do
```

The `sudo docker network ls --format '{{.Name}}'` is not a Linux shell script thing but something that comes from Docker. Docker is programmed in Go and Go programmers probably recognize it. As ChatGPT explains: _--format '{{.Name}}' uses Go template syntax to extract the Name field from Docker’s internal JSON-like structure — it has nothing to do with the visible column headers in the regular docker network ls table._ More info in the [Docker documentation](https://docs.docker.com/engine/cli/formatting/).

Another fancy one:

```
sudo docker network inspect "$netwrk" | jq -r '.[0].Containers | to_entries[] | .value.Name'
```

Here jq is being utilized, it handles json and needs to be installed. `docker inspect` returns a json string with tons of information and jq helps to extract from it the fields that you want. I use it extensively here to extract network names, restarting policy and information on the first mounted directory.

My output for the Docker section on my server currently looks like this:

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-10-06 10:00:18 UTC; 1 month 1 day ago

Note: output here depends on Docker running and containers being started.

CONTAINER ID   IMAGE                               COMMAND                  CREATED        STATUS        PORTS                                         NAMES
679a2113e093   postgres:16                         "docker-entrypoint.s…"   18 hours ago   Up 18 hours   0.0.0.0:5678->5432/tcp, [::]:5678->5432/tcp   mijnwoonplaatscontainertestdb
09290f3a0333   geert679/mijnwoonplaatsapp:latest   "java -jar app.jar"      18 hours ago   Up 18 hours   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp     mijnwoonplaatscontainerapp
5c640227c5f2   postgres:16                         "docker-entrypoint.s…"   24 hours ago   Up 24 hours   5432/tcp                                      mijnwoonplaatscontainerproductiondb

NETWORK ID     NAME                    DRIVER    SCOPE
2664b1f7c54e   bridge                  bridge    local
db6545035f33   host                    host      local
188fc32b7b93   mijnwoonplaatsnetwork   bridge    local
42cba09f71d6   none                    null      local

Network mijnwoonplaatsnetwork includes the following containers (only shown if running):
  • mijnwoonplaatscontainerapp
  • mijnwoonplaatscontainerproductiondb

CONTAINER DETAILS
▪ mijnwoonplaatscontainertestdb:
  Network: bridge
  RestartPolicy: unless-stopped
  Mounts (only first):
    Type: volume
    Name: pgdatamijnwoonplaatsdbtest
    Source: /var/lib/docker/volumes/pgdatamijnwoonplaatsdbtest/_data
    Destination: /var/lib/postgresql/data

▪ mijnwoonplaatscontainerapp:
  Network: mijnwoonplaatsnetwork
  RestartPolicy: unless-stopped
  Mounts (only first):
    Type: null
    Name: null
    Source: null
    Destination: null

▪ mijnwoonplaatscontainerproductiondb:
  Network: mijnwoonplaatsnetwork
  RestartPolicy: unless-stopped
  Mounts (only first):
    Type: volume
    Name: pgdatamijnwoonplaatsdbproduction
    Source: /var/lib/docker/volumes/pgdatamijnwoonplaatsdbproduction/_data
    Destination: /var/lib/postgresql/data
```

I think it is actually great. Don't abuse it, I am exposing too much.





