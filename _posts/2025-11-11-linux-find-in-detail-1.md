# Linux find in detail part 1

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

## Test categories

I did a bit of categorizing to get the test expressions sorted. 

###  Name and path

These test expressions care about name and/or the full path of the file:

### **-name**



