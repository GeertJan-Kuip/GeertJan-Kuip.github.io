# Linux in-depth 1

This is the first of a series to learn more about Linux, the way it deals with processes, users and filesystems. My concrete aim is to be able to log in using ssh on any Linux server and figure out what's going on there.

## File system and navigation

### About root, '/' and '/root'

A Linux machine has a root user with all permissions. This special status goes along with a folder '/root' to which only this superuser has access. This folder can be called the 'root' folder but this can be misleading. The folder one level higher, '/', actually deserves more to be called root. One characteristic of both folders is that only the superuser is allowed to create directories within them or to access them. Another typical thing is that when you are as superuser in /root, bash displays '~#' instead of '/root#'. It is the '~' that indicates you are in root, if you are in '/' you see '/#'. What I learned from the forums is that superusers are supposed to store their critical files in '/root' so that other users will not see or touch them.

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

### Changing permissions

The command to change permissions is chmod in the form of `chmod {options} filename`. Explanation can be found on the [Ubuntu website](https://help.ubuntu.com/community/FilePermissions). You can change permissions using a system with letters or a system with numbers.

### File  ownership









 