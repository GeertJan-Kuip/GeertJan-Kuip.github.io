# Linux in-depth 1

This is the first of a series to learn more about Linux, the way it deals with processes, users and filesystems. My concrete aim is to be able to log in using ssh on any Linux server and figure out what's going on there.

## File system, permissions and navigation

### About root, `/` and `/root`

A Linux machine has a root user with all permissions. This special status goes along with a folder `/root` to which only this superuser has access. This folder can be called the 'root' folder but this can be misleading. The folder one level higher, '/', actually deserves more to be called root. One characteristic of both folders is that only the superuser is allowed to create directories within them or to access them. Another typical thing is that when you are as superuser in /root, bash displays `~#` instead of `/root#`. It is the `~` that indicates you are in root, if you are in `/` you see `/#`. What I learned from the forums is that superusers are supposed to store their critical files in `/root` so that other users will not see or touch them.

### The . and .. in directories

If you use the command `ls -a` or `ls -al` on some directory, you will always see two items `.` and `..` in the listing. It is the -a that triggers their appearance and they stand for the current directory and the parent directory. This is Unix legacy and you do not have to break your head over it. It is consistent with the commands `cd .` and `cd ..`.

### ls

The list command ls is important to check what is in a directory. There are tons of flags helpful in getting the output and output format you want. The most relevant is --help, which lists all possible flags and their use:

|flag|what it does|
|----|----|
|--help|lists all flags + explanation|
|-a|all files, including hidden ones, are shown|
|-l|files + folders shown as detailed list|
|-t|sort by last modification time|
|-r|reverse order|
|*.txt|show only txt files|

I can add more but I will forget them anyway. Better use --help.

### Hidden files starting with .

File- and directory names starting with a dot are hidden, which doesn't mean that they shouldn't be seen but that they are supposed not to be of interest to the user. They do not show up in listings unless you use flag -a or -A. I cam e across a post of someone complaining that programmers overuse this style of file/directory naming.

### The meaning of ~

In Linux, `cd ~` brings you to your home directory. For the superuser this is `/root` and it explains why you see `'~#'` when in this directory. 

if a file name ends with tilde, it is a backuop file of some program.

### Permissions

If you list the contents of a directory with `ls -l` you see each line starting with a 10 character code. At least in Ubuntu. The first character is either d, - or l, whereby 'd' denotes a directory, '-' a file and 'l' a symbolic link.

Subsequently there are three groups of three, representing permissions for the owner, users in the same group as the owner, and all other users.

In each group of three the charactor r indicates read permission, w is write permission and x is permission to execute the file.

Example: `-rwxr-xr-x` is a file, owner has all permissions, others can read and execute but not write.

### Changing permissions with `chmod`

The command to change permissions is chmod in the form of `chmod {options} filename`. Explanation can be found on the [Ubuntu website](https://help.ubuntu.com/community/FilePermissions). You can change permissions using a system with letters or a system with numbers.

### File  ownership and `chown`

When you list files using `ls -l` you see two columns that seem to represent a user. The first one denotes the owner of the file/directory, the second the group. Every file/directory is owned by both a user and a group. In the example below, the first root is the user that owns the folder, the second is the name of the owning group.

```
drwxr-xr-x   2 root root  4096 Aug 27  2024 media
```

The command `chown` makes it possible to change ownership of a group. You can change both the user that owns the file or directory, and the group that owns the file/directory:

```
sudo chown vivek demo.txt  // vivek will be the user owning demo.txt
 
sudo chown vivek:vivek demo.txt  // demo.txt owned by user vivek and group vivek
```

It is possibly changing ownership for all nested content of a directory using flag -R (recursive). This is strongly warned against on the fora, as it can mess up the whole OS.

### The /etc folder

This folder is on the root and contains the text files that are about groups, users and their privileges. The relevant files are:

#### `/etc/passwd`

In this textfile you find the users. You can get with `cat /etc/passwd`, which gives the contents of the file. Besides the 'real' human users, all sorts of installed services are also listed. A typical line for them is:

```
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
or
pollinate:x:102:1::/var/cache/pollinate:/bin/false
```

The `nologin` or `false` element indicates that they do not have a bash terminal, which is logical given their status as service. The first number is the userID, the second the groupID, then comes name/description. The first path after that is the home directory and the last path is the path of the login shell.

PostgreSQL is a special one among the applications in the list. Here my example:

```
postgres:x:108:111:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
```

UserID is 108, groupID 111, postgres created and owns `/var/lib/postgresql` and `/bin/bash` is where the login shell resides. It means that you can work in your terminal as `postgres@geertjan-server:~$`. The command to do so is `sudo -i -u postgres`.

Note that the tilde in `postgres@geertjan-server:~$` indicates that you are in the postgres home directory. Giving the command `ls -lR` immediately shows you all the nested that postgres has stored here (all files belonging to a database in my case).

The last case is that of real users. There is the real, default root user and there are the new users you create.

```
root:x:0:0:root:/root:/bin/bash

geertjan:x:1000:1000:Geert-Jan,,,:/home/geertjan:/bin/bash
```

Note that any new user has `/bin/bash` as last element, meaning that they all work with the same terminal. Users that are not root have a folder with their username in the /home directory that they own (like `/home/geertjan`). If you want a specific user printyed out and not the whole contents of passwd, type:  

```
cat /etc/passwd | grep -i geertjan
```

#### `/etc/group`

Every user belongs to a group. To see in which group, you can type `id {username}` which gives you something like this:

```
uid=1000(geertjan) gid=1000(geertjan) groups=1000(geertjan),27(sudo),100(users)
```

Upon creation of the user the similarly named group was created, and to get geertjan in the sudo group the following command was used:

```
usermod -aG sudo geertjan
```

Note that sudo is missing, I created this change logged in as the root user.


A user named geertjan has userid 1000 and is in groups 'geertjan' and group 'sudo'. 

#### `/etc/sudoers`

```
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        use_pty

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

@includedir /etc/sudoers.d
```

### The `sudo` command

To be able to use the sudo command you must have been granted the privilege, meaning you are added to the sudoers file. This file lives in /etc/sudoers and you need root permission to access it.





 