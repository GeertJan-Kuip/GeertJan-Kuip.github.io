# Linux inode, hard links and symbolic links

I stumbled upon a [YouTube video](https://www.youtube.com/watch?v=ScDv02ff8oc) from [Dave](https://ysap.sh/) and he discussed the workings of Unix/Linux filesystems. In the previous post I noticed that hard lilnks and symbolic links seem to have quite some relevance in Linux and Dave explains what's behind it. This post is a summary of the content of his video.

## What an inode is

For every file, no matter the type, Linux stores three things. There is the path, like `home/geert/scripts/myscript.sh` or `/var/lib`, then there is the inode, which contains the metadata but **not** the path or name of the file, and there is the actual content of the file. The path points to the inode and the inode points to the content/data. 

### Using `stat` to get Inode data

The inode metadata of a regular file, a directory, a symbolic link can be inquired using the `stat` command (Dave uses gstat but that one requires extra installation on my Ubuntu server):

```
  File: kort.sh
  Size: 583             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 255503      Links: 1
Access: (0775/-rwxrwxr-x)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-11-11 20:36:03.914489922 +0000
Modify: 2025-10-06 21:09:53.871102095 +0000
Change: 2025-10-06 21:09:53.871102095 +0000
 Birth: 2025-10-06 20:06:15.029554543 +0000
```

Using stat on a directory gives this:

```
  File: scripts
  Size: 4096            Blocks: 8          IO Block: 4096   directory
Device: 8,1     Inode: 383256      Links: 2
Access: (0775/drwxrwxr-x)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-11-12 16:02:30.400431618 +0000
Modify: 2025-10-14 14:47:59.548172010 +0000
Change: 2025-10-14 14:47:59.548172010 +0000
 Birth: 2025-10-13 15:11:26.200142446 +0000
```

Using stat on a symbolic (soft) link gives this:

```
  File: mysymboliclink -> file1.txt
  Size: 9               Blocks: 0          IO Block: 4096   symbolic link
Device: 8,1     Inode: 524600      Links: 1
Access: (0777/lrwxrwxrwx)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-11-13 19:15:07.964199821 +0000
Modify: 2025-11-13 19:15:06.114213728 +0000
Change: 2025-11-13 19:15:06.114213728 +0000
 Birth: 2025-11-13 19:15:06.114213728 +0000
```

What you see here is that files, directories and symbolic links give very similar output. Each has its own unique Inode number and there is a field that tells what type it is (there are more types than this). Furthermore, each of these three files has the same 'Device' value which indicates that they live on the same filesystem. This `8,1` value is sometimes represented as a large integer, Dave explains why these different forms are actually representing the same value. 

A remarkable thing is that the 'Size' value equals the amount of characters in the contents of the regular file (so kort.sh has 583 characters) or, in the case of the symbolic link, the amount of characters of the file that the link points to (9 in this case). Because the symbolic link and its target reside in the same folder the full path is not included, but when they live in different directories it needs more bytes.

It is important to note that while the File field in the output suggests that the path/name of the file is part of the metadata, that is definitely not the case. 

### What the 'Links' value represents

Every filename or path points to an inode. This is also true for symbolic links. All three types have their own inode and their own datablocks. The 'Links' value tells you how many filenames/paths point to the same inode. As symbolic lilnks do not point to the inode of the file the refer to but to its path, the value of Links of a regular file will not increase when you create symbolic links to that file. Those symbolic links, even though they refer to another file, have their own inode and their own datablock.

### Why the directory type has 2 links

The value of 'Links' for the 'scripts' directory has a value of 2 for links. Where does this come from? It is simple, every directory contains a `.` hidden link that points to the directory itself. This means that there exists an extra hidden link to every directory. Every directory therefore has at least a value of 2 for 'Links'. Most directories have even more, because every subdirectory has the `..` link that also points to it. As you can see below, the root folder has 23 links, which corresponds to the number of subfolders plus 2 (both `.` and `..` point to it, the latter because there is nothing beneath root anymore).

```
  File: /
  Size: 4096            Blocks: 8          IO Block: 4096   directory
Device: 8,1     Inode: 2           Links: 23
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-11-12 16:02:30.087432934 +0000
Modify: 2025-10-06 10:00:15.206000000 +0000
Change: 2025-10-06 10:00:15.206000000 +0000
 Birth: 2025-08-14 11:02:01.000000000 +0000
```

### What a hard link is

Now that we know how a symbolic or soft link works, and now that we have the example of the directory type that have an inode that is pointed to by multiple files, it is easier to understand what a hard lilnk is. A hard link is a regular file that points to the same inode, and therefore the same content, as another file. In Unix/Linux you can create such hard links yourself with command `ln <target> <link_name>`. If you remove either of the files, nothing happens to the underlying inode or datablocks, those will only be removed when no link points to the inode anymore. A sort of garbage collector is at work here.

As an example, I did the following:

```
geertjan@geertjan-server:~$ mkdir temp  			# new folder
geertjan@geertjan-server:~$ cd temp     			# get into it
geertjan@geertjan-server:~/temp$ echo 'Hello' > file1.txt	# create file size 5 (5 characters)
geertjan@geertjan-server:~/temp$ ln file1.txt file2.txt		# create file2.txt as a hard link
geertjan@geertjan-server:~/temp$ stat *				# show inode data of bothe files
  File: file1.txt
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 524599      Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-11-13 20:04:11.809287741 +0000
Modify: 2025-11-13 20:04:11.809287741 +0000
Change: 2025-11-13 20:04:26.738225668 +0000
 Birth: 2025-11-13 20:04:11.809287741 +0000
  File: file2.txt
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 524599      Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/geertjan)   Gid: ( 1000/geertjan)
Access: 2025-11-13 20:04:11.809287741 +0000
Modify: 2025-11-13 20:04:11.809287741 +0000
Change: 2025-11-13 20:04:26.738225668 +0000
 Birth: 2025-11-13 20:04:11.809287741 +0000
```

As you  see, both files point to the same inode, number 524599. There are 2 links (they both link to this inode) and the size is 6 (5 characters + newline). Both files are identical except for their filename, but this filename is only stored in the directory file `/home/geertjan/temp`. The contents of this file can be retrieved with a simple `ls` command. 

That they are both the same file can be seen by changing file2.txt and then open file1.txt:

```
geertjan@geertjan-server:~/temp$ echo 'My name is Geert-Jan' > file2.txt
geertjan@geertjan-server:~/temp$ cat file1.txt
My name is Geert-Jan
```

### Timestamps

Unix/Linux stores different timestamps for creation, last modification, last change and last access. The difference between modification and change is that modification is about the datablocks while change is about the inode. The interesting thing is that, because the size of a file is stored in inode, every modification that results in a change in size, will also update the change timestamp. Dave remarked that he doesn't fully trust the timestamps and that he thinks they are quite expensive, because they need to be updated so often. The creation ('Birth') timestamp, which I considered relevant myself, does not have a flag or method in the find method, which I find odd but it might be that this timestamp is either not always available or simply distrusted.





