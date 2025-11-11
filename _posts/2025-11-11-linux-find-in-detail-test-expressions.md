# Linux find in detail - test expressions

In a previous post I discussed the use of find, in this post I will get into more detail. Based on the Linux [man page](https://www.man7.org/linux/man-pages/man1/find.1.html) I will specifically discuss the so called 'tests', which is the subset of expressions that determines what files will be found. In a next post I will discuss the 'actions', which is the subset of expressions that tell Linux what to do with the files and directories being found.

## File metadata

Linux stores metadata on files. Knowing the fields of this metadata gives a good clue on what tests are available under the find command. To get the metadata, you can use the `stat` command:

```
geertjan@geertjan-server:~$ stat kort.sh
  File: kort.sh
  Size: 583             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 255503      Links: 1
Access: (0775/-rwxrwxr-x)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-10-08 09:40:17.276199712 +0000
Modify: 2025-10-06 21:09:53.871102095 +0000
Change: 2025-10-06 21:09:53.871102095 +0000
 Birth: 2025-10-06 20:06:15.029554543 +0000
```

The fields that Linux keeps track of are included in the response when you are inquiring a file. You can adjust the command to `stat kort.sh -f`, adding the -f flag. In this case you get information on the filesystem in which kort.sh resides:

```
geertjan@geertjan-server:~$ stat kort.sh -f
  File: "kort.sh"
    ID: ea7dcdc7ccb5d7d6 Namelen: 255     Type: ext2/ext3
Block size: 4096       Fundamental block size: 4096
Blocks: Total: 9759259    Free: 8166616    Available: 7755436
Inodes: Total: 2427136    Free: 2334083
```

All the fields you see above, or at least most of them, are translated in one or more test expressions. The one missing in the metadata above is _path_, but of course you can test on that as well.

## File types

The three most common file types are regular file, directory and symbolic link. But there are a few more and you can test on these types using the -type c expression:

- block (buffered) special
- character (unbuffered) special
- named pipe (FIFO)
- socket
- door (Solaris)

## Symbolic links

Symbolic links play a large role in the find command, there are many flags that help determine the way symbolic links are handled. The most important are -H, -L and -P, which immediately come after the 'find' command. -P is the default and means that symbolic links are never followed. 

Among the test espressions, we find a couple of tests that specifically care about symbolic links, namely -lname, -ilname, -type l,  -xtype l.

##  General syntax

The structure of a find command, according to the man page, is the following:

```
       find [-H] [-L] [-P] [-D debugopts] [-Olevel] [starting-point...]
       [expression]
        .
```

As you see, the expressions come at the end and are proceeded with options related to the treatment of symbolic links (-H, -L, -P), the -D debugoptions and the -Olevel option that has to do with the under-the-hood optimization process. In examples you hardly see any of these options flags being utilized.

'Starting point' is the one that most of the times follows the 'find' command. You can specify a single directory, using `.`  for the current one, or a list of directories, separated by whitespaces. 

Under 'expression' different categories exist, namely 'tests', 'actions', 'global options', 'positional options' and 'operators'. Here we focus on tests but there are some relevant global and positional options that will be discussed at the end.

## Basic globbing, extended globbing, regular expressions

This is a somewhat confusing topic. If you want to use wildcards or regex in your tests be aware of the following:

- Only -regex uses regular expressions
- The default regex syntax is _Emacs Regular Expresions_
- This default can be changed using the -regextype option
- For -name and other tests, _basic globbing_ is being used. 
- You can turn on _extended globbing_ with command `shopt -s extglob` but:
- Test expressions in _find_ will still use basic globbing, so it doesn't matter

### What basic globbing is:

ChatGPT provided this:

| Symbol    | Meaning                            | Example |
|------------|------------------------------------|----------|
| `*`        | Matches any string (including empty) | `*.txt` matches `a.txt`, `foo.txt` |
| `?`        | Matches exactly one character        | `?.txt` matches `a.txt` but not `ab.txt` |
| `[abc]`    | Matches one of the listed characters | `[ch]at` matches `cat`, `hat` |
| `[a-z]`    | Character range                      | `[0-9].log` matches `1.log` … `9.log` |
| `[!abc]`   | Negated character class              | `[!0-9]*` matches anything not starting with a digit |

### Extended globbing

Not relevant here as the test expressions do not use it, but good to know what it is and how it differs from basic globbing and regular expressions:

| Pattern      | Meaning | Example |
|---------------|----------|----------|
| `?(pattern)`  | Matches **zero or one** occurrence of `pattern` | `?(foo).txt` matches `foo.txt` or `.txt` |
| `*(pattern)`  | Matches **zero or more** occurrences of `pattern` | `*(foo).txt` matches `.txt`, `foo.txt`, `foofoo.txt` |
| `+(pattern)`  | Matches **one or more** occurrences of `pattern` | `+(foo).txt` matches `foo.txt`, `foofoo.txt` |
| `@(pattern)`  | Matches **exactly one** of the given patterns separated by `|` | `@(foo|bar).txt` matches `foo.txt` or `bar.txt` |
| `!(pattern)`  | Matches **anything except** the given pattern(s) | `!(foo).txt` matches all `.txt` files except `foo.txt` |


## All tests categorized

I did a bit of categorizing to get the test expressions sorted. 

###  Category 1 - Name and path

These test expressions care about name and/or the full path of the file:

#### -name _pattern_

The -name test only tests the filename, not the path. It uses the bash shell pattern matching ('basic globbing'), not the more extensive regex pattern. You will use '*' most of the times, but you can use ? or something lilke [a-z0-9] as well. Example:

```
~$ find . -name '?[a-z][r]??sh'
./kort.sh
```

#### -iname _pattern_

Like -name but case insensitive

#### -lname _pattern_ 

Like name but only returns symbolic links. Doesn't go well with the -L flag.

#### -ilname _pattern_

Like -iname but only returns symbolic links, doesn't go well with the -L flag.

#### -path _pattern_

Like -name and its derivatives, -path uses basic globbing. It examines the whole path, and `.` and `/` have their literal meaning so you can use them as characters. 

The path to be examined is the path that starts at one of the starting points of the find command. Unless the starting point is '/', you will not examine the whole absolute path. 

Instead of -path you can use -wholename, but this alternative is 'less portable' according to the docs.

#### -ipath _pattern_

Same as -path but case-insensitive.

#### -regex _pattern_

Like path, but you use an Emacs type regular expression. This provides extra possibilities at the cost of a more extensive syntax. The matching is done on the whole path, not just on the file name.

#### -iregex _pattern_

Same as -regex but case-insensitive.

### Category 2 - Ownership

You can select files and directories based on ownership with the following expressions:

#### -user _uname_

Selects items owned by specified user. You can either use username or userid.

#### -uid _n_

Specify the user id and use + as prefix to select all users with a higher uid, - to select all users with a lower id and neither - or + to select the user that has exactly the given id.

#### -group _groupname_

Selects items owned by specified group. You can either use groupname or group id.

#### -gid _n_

Similar to -uid, use + and - to indicate higher/lower than.

#### -nouser

No user corresponds to file's numeric user ID.

#### -nogroup

No group corresponds to file's numeric group ID.

### Category 3 - Timestamps

Linux keeps track of time/date of creation, last modification, last access and last change. 'Change' in this context means change of metadata/status, not a change in the contents of the file.

Test expressions do either one of two things. They check if the file is older/newer/equally old based on a numeric input you give, whereby you either specify the amount of minutes or the amount of days. Or they check if a timestamp of a file is either less recent or more recent than some timestamp on a reference file you provide as argument.

In the case of numeric input, you can set a flag (it is a positional option so it must preceed the expression where it is applied) named _-daystart_. If -daystart is being used, time is measured starting at beginning of today rather than 24 hours ago.

When you check if a timestamp is before or after a certain 'past minutes' or 'past days', you use the + or - prefix on the numeric input. Not using any of these means that you are checking for the exact timestamp of a file.

#### -newer _reference_

Requires a argument that is a valid file, the so-called reference file. Returns true for files with a modification timestamp more recent than that of the reference file.

#### -newerXY _reference_

More extensive variant. X stands for a timestamp of the file to be examined, Y stands for a timestamp of the file being reference. To indicate which timestamp to inspect use the following letters:

- a - last access time
- B - birth time (creation)
- c - last status change time
- m - last modification time
- t - only to be used for Y. If used, provide a timestamp as reference and not a file

#### -anewer _reference_

Tests if the time of last access of the tested file is more recent than the last modification time of the reference file. Bit odd comparison I would say.

#### -cnewer _reference_

Tests if the time of last status change of the tested file is more recent than the last modification time of the reference file. Equally far-fetched as the previous I would say.

#### -amin _n_

File was last accessed less than, more than or exactly n minutes ago. If +n, more than, if -n less than, if n, exactly.

#### -atime _n_

File was last accessed less than, more than or exactly n*24 hours ago. Be aware that fractional parts are ignored, so -amin +1 means the file nust have been accessed at least two days ago.

#### -cmin _n_

Same as -amin but now it measures change time (which is different from modification time).

#### -ctime _n_

Same as -atime, but applied to change timestamp.

#### -mmin _n_

Same as -amin but now it measures modification time.

#### -mtime _n_

Same as -atime, but applied to modification timestamp.

#### -used _n_

File was last accessed less than, more than or exactly n days after its status was last changed.

### Category 4 - Permissions

Every file has permissions, which can be written in different forms. The stat command provides the numeric or octal form (0775) and the string form (-rwxrwxr-x):

```
Access: (0775/-rwxrwxr-x)
```

There is also the symbolic form in which you set persmissions using the letters u (user that owns the file), g (group that owns the file), o (other users), a (all users), r (read), w (write) and x (execute). You can add or remove permissions on a file with statements like `u+x`, `g-w`, `+r`, `go=rq` and `u+x,go=rx`.

You can check permissions in an absolute way with `-perm`, which means examining the access 'codes'. It is also possible to examine access permissions based on the current user, using the `-readable`, `-writable` and `-executable` tests.

#### -perm _mode_

Here the permission bits of the file must exactly match the permission bits of the argument. You can use both the octal mode (0774) and the symbolic mode. The two examples below are identiacal and only return true if the permssion string is ?----w----:

```
-perm g=w
-perm 0020
```

#### -perm _-mode_

You can use both octal and symbolic mode, test passes if the indicated permissions are set, while all permissions that are not set in the argument are allowed to vary.

#### - perm _/mode_

This is the least strict way of using -perm. Test passes if only one of the three (owner, group, other user) has the permission that is indicated in the argument. The amn page provides this example:

```
# Search for files which are writable by either their owner or their group.
$ find . -perm /220
$ find . -perm /u+w,g+w
$ find . -perm /u=w,g=w
```

#### -readable

File can be read by user.

#### -writable 

File can be written by user.

#### -executable

File can be executed by user.

### Category 6 - Types

You can check type with -type and -xtype. They differ in their handling of symbolic links. 

#### -type _c_

The argument c can be one of the following:

- b - block (buffered) special
- c - character (unbuffered) special
- d - directory
- p - named pipe (FIFO)
- f - regular file
- l - symbolic link
- s - socket
- D - door (Solaris). I have no clue what this is.

#### -xtype _c_

Like -type but different handling of symbolic links.

#### -fstype _type_

Returns true if the file is on a filesystem of type _type_.

### Category 7 - File size

#### -empty

Returns true if file or directory is empty.

#### -size _n[cwbkMG]_

File uses less than, more than or exactly n units of space, rounding up.

- b: 512-byte blocks
- c: bytes
- w: two-byte words
- k: kibibytes (KiB, units of 1024 bytes)
- M: mebibytes (MiB, megabytes)
- G: gibibytes (GiB, gigabytes)

### Category 8 - Miscellaneous

#### -links _n_

File has more than, less than or exactly n hard links. A hard link is a file that directly points to the same inode (the files data on disk) as the original file. It is thus not a symbolic link.

#### -samefile _name_

File refers to the same inode as name. I do not know how and when this will happen, apart from symbolic links.

#### -context _pattern_

(SELinux only) Security context of the file matches glob pattern. Cannot explain what is meant by this.

