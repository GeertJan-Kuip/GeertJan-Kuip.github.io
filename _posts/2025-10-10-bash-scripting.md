# Bash scripting

I am writing a Linux bash scripts that I can run on a server or my own machine. Its purpose is to provide terminal output that immediately tells me about what is installed and running on that machine regarding the following:

- Nginx (how is it configured)
- Docker (which networks and containers are running and what is their setup)
- Filesystem (what is to be found in the home directory)
- PostgreSQL (is it root installed and if so, what databases, what tables, what ownership/permissions)

As bash script language is rather obscure, I publish here some advice I got from ChatGPT, some pitfalls I might encounter next time, and some work-in-progress scripts that contain working code.

## ChatGPT advice

Here is a small block with general advice. Working with variables is painfull as the script language does not always make a difference between newline and space, following the guidelines below saves a lot of trouble. Furthermore the syntax of if-statements requires precision. Use `[[` double square brackets and leave spaces between things.

```
#!/bin/bash
set -euo pipefail   # safer defaults
IFS=$'\n\t'         # strict input field splitting

# Always quote variables
echo "Hello, ${USER}"

# Use [[ ]] for tests, (with spaces around them)
if [[ "$name" == "Geert-Jan" ]]; then
    echo "Welcome back!"
fi

# Use functions
log() { echo "[LOG] $*"; }

# Use shellcheck
# -> a lint tool that points out errors and unsafe syntax
```

Few more advices:

- No spaces when assigning variables. `MYVAR="Hello"` instead of `MYVAR = "HELLO".
- Tests can be multiple using `&&` and/or `||`
- If you use `[[ ]]` for tests you can use `==` and `!=`.
- jq is a library that helps to work with json output. Use it in pipes. My code shows examples and there is documentation.
- shellcheck is not that good
- Start nano editor with the -m flag, it lets you use your mouse to put the cursor anywhere.
- To copy paste in nano, do cut-paste-paste (first paste to put the line back, second for the copy).
- Docker commands provide output options with the --format flag. See Docker reference.
- If you want styling of text using Unicod characters, use the -e flag in the echo command.
- Variables can be used in commands, like `"${MyVariable}"`
- Capitals are not required for variable names.

To assign a multiline string to a variable, for example if you want to create a long query string, this is a safe way:

```
QUERY=$(cat <<'EOF'
Just some string
that contains multiple 
lines
EOF
)
```

## Sample code

The code below creates text output concerning Docker containers and networks.

```
set -euo pipefail
IFS=$'\n\t'

BLUEYELLOW='\e[44;93m'
RESETCOLOR='\e[0m'

echo ""
echo -e "${BLUEYELLOW}                                                                DOCKER                                                                                             ${RESETCOLOR}"
echo ""

docker ps --all
echo ""

QUERY2=$(cat <<'EOF'
SELECT * FROM buurten LIMIT 5;
EOF
)

do_query () {
  docker exec -it mijnwoonplaatsdbcontainer psql -U user01 -d db01 -At -c "$1"
}

#do_query "${QUERY2}"  // I disabled this one but you can do psql db query like this

docker network ls
echo""

for netwrk in $(docker network ls --format '{{.Name}}'); do
  if [[ "$netwrk" != "host" && "$netwrk" != "bridge" && "$netwrk" != "none" ]]; then
    echo -e "Network \e[4;37m$netwrk\e[0m includes the following containers:"
    for name in $(docker network inspect "$netwrk" | jq -r '.[0].Containers | to_entries[] | .value.Name'); do
      echo -e "  \u2022 $name"
    done
  fi
done

echo ""
echo "CONTAINER DETAILS"

for container in $(docker ps --format '{{.Names}}'); do
  echo -e "\u25AA \e[4;37m$container:\e[0m"
  #echo "$container:"
  NETWORKNAME=$(docker inspect "$container" | jq -r '.[0].HostConfig.NetworkMode')
  RESTARTPOLICY=$(docker inspect "$container" | jq -r '.[0].HostConfig.RestartPolicy.Name')
  echo "  Network: ${NETWORKNAME}"
  echo "  RestartPolicy: ${RESTARTPOLICY}"

  MOUNT_TYPE=$(docker inspect "$container" | jq -r '.[0].Mounts.[0].Type')
  MOUNT_NAME=$(docker inspect "$container" | jq -r '.[0].Mounts.[0].Name')
  MOUNT_SOURCE=$(docker inspect "$container" | jq -r '.[0].Mounts.[0].Source')
  MOUNT_DESTINATION=$(docker inspect "$container" | jq -r '.[0].Mounts.[0].Destination')
  echo "  Mounts (only first):"
  echo "    Type: ${MOUNT_TYPE}"
  echo "    Name: ${MOUNT_NAME}"
  echo "    Source: ${MOUNT_SOURCE}"
  echo "    Destination: ${MOUNT_DESTINATION}"
  echo ""
done
```




