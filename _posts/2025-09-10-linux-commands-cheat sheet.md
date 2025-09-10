# Linux commands cheat sheet

Briljant [video of NetwerkChuck](https://www.youtube.com/watch?v=gd7BXuUQ91w) turned into a cheat sheet. I have added links to pages where flags are explained.

## General

- ssh {username@ipaddress} _- login to server_
- clear
- apt update _- update all repositories. requires sudo_
- apt install {packagename} _- install package_
- man {command name} _- info about command_
- whatis {command name} _- very brief info about command_
- which {command name} _- location of command_
- whereis {command name} _- location of command, provides all locations_
- uname -a _- info on operating system_
- free _- amount of free memory_
- df -H _- disk space info_
- history _- list of previous commands_

## Users, groups, permissions

- whoami
- useradd {username} _- add user. requires sudo_
- adduser {username) _- add user and set all params_
- su {username} _- switch user_
- exit _- back to previous user_
- passwd {username} _- change password. requires sudo_
- passwd _- change own password_
- finger {username} _- inspect user. finger must be installed_
- usermod {flag} {value} {user} _- modify user properties. [link](https://www.geeksforgeeks.org/linux-unix/usermod-command-in-linux-with-examples/)_
- usermod -aG sudo {username} _- add user to sudo group. requires sudo permissions_


## Processes

- ps -aux _- list of processes_
- ps -aux | grep {nameofprocess} _- list specific process
- top _- live list of processes_
- htop _- same as top, looks nicer_
- kill -9 {PID} _- kill process with specified ID_
- pkill -f {processname} _- kill process with specified name_
- systemctl start {processname} _- starts process_
- systemctl stop {processname} _- stops process_
- systemctl status {processname} _- status of process_
- systemctl restart {processname} _- restart process_
- systemctl enable {processname} _- process will start automatically on startup_
- systemctl disable {processname} _- process won't start automatically on startup_
- systemctl daemon-reload _- soft reloading, required for changes in service files to take effect after restart_
- nohup {command} _- chosen command will keep running after logout/closing terminal_
- {command} & _- command will run in background

## Internet

- wget {url} _- get contents of url and stores it in current dir under url last element name_
- curl {url} > {filename} _- same_
- ifconfig _- find ip address_
- ip address _- find ip address_
- ip address | grep eth0 _- only see eth0 address (sort of)_
- ip address | grep eth0 | grep inet | awk '{print $2}' _- really only the ip address_
- cat /etc/resolv.conf _- find DNS info_
- resolvectl status _- find current DNS server_
- ping {url} _- see if website is up. use ctrl c to stop it_
- ping {url} {integer n} _- ping url n times_
- traceroute {url} _- displays the route taken by the response, all hubs_
- netstat _- list of all ports on machine. [link](https://www.geeksforgeeks.org/linux-unix/netstat-command-linux/)_
- netstat -tulpn _- more relevant list. only listening tcp and udp with info about process involved_
- ss _- socket statistics, similar to netstat, can use -tulpn as well_
- ufw allow {port number} _- requires sudo. allows traffic on specified port_
- ufw enable _- port from previous command now works_

## Files

- ls (-l, -al)  _- list files_
- pwd  _- print working directory_
- cd  _- change directory_
- touch {filename}  _- create file_
- echo {text} {> filename}  _- print to console or add text to file_
- nano {filename}  _- create/open and edit file. exit by ctrl x, type y for save_
- vim {filename} _- create/open and edit file. i to start edit, escape :wq to save and exit_
- cat {filename} _- read file_
- cat {filename} | sort _- read file and sort output lines_
- less {filename} _- read file page by page_
- head { filename} _- read begin of file_
- tail {filename} _- read end of file}
- stat {filename} _file metadata_
- cmp {file1} {file2} _- compare two files_
- diff {file1} {file2} _- precise descriptions of differences between two files_
- shred {filename} _- overwrite file so it becomes unrecoverable_
- mkdir {directory name} _- create directory_
- cp {source} {destination} _- copy file_
- mv {source} {destination} _- move file_
- rm {file} _- remove file_
- rmdir (-r) {directory} _- remove directory. -r removes recursively_
- ln (-s) {file} {link} _- create link to a file_
- source {filename} _executes commands in file_
- zip {filename zipfile} {filename original} _- zip_
- unzip {filename zipfile} _- unzip, you get options if file exists_
- find {directory} (-name, -type, -perm etc) {expression} _- [link](https://www.redhat.com/en/blog/linux-find-command) and [another link](https://help.ubuntu.com/community/find)_
- chmod {change formula} {filename} _- "change mode", change file permissions. [link](https://www.digitalocean.com/community/tutorials/how-to-set-permissions-linux)_
- chown {options} {user/group} {file} _- change ownership of file. [link](https://linuxize.com/post/linux-chown-command/)_


