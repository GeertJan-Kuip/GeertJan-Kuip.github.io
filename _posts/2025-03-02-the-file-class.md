## The File class


The File class is part of Java's IO package and some detailed knowledge about it is required for 1Z0-819. I studied the source file and chapter 19 of the book, these are my notes.

### Absolute and relative paths

Absolute path is the full path of a file or directory. In UNIX systems the root of it is denoted by '/', in Windows it is denoted by either 'C:\' (C can be replaced with any other drive name) or by '\\\\' or '\\\\\\\\'. Relative path is the path starting from the current directory. To get the current directory you can use these Java commands:

```
System.getProperty("user.dir");
```

To get the default separator of your operating system, use:
```
System.out.println(System.getProperty("file.separator"));
```

For Windows it is '\\', Unix-based systems use '/'. The value of the File class is that it contains an abstract form of the path so you don't have to worry about writing the path in accordance with some specific operating system in mind.

### Files and directories are one kind

For Java, a file and a directory are sort of the same thing, you can rename them using the same method for example. Note that a file can miss an extension which will make it look like a directory. A directory can contain a dot. Don't get mislead by this.

### Construccting a File object

The File class can be constructed in multiple ways, there are four public variants of the constructor. The simplest is the one in which you provide a string representing the absolute path. Two other constructors allow you to provide two arguments, one for the parent part and one for the child part of the absolute path. The fourth one requires an URI.

Note that you can create a File object of a file/directory that doesn't exist. The path in a File object doesn't have to denote something real. The exists() method checks for the existence of a file- or directory path.

### Methods

File has quite a lot of methods available. The book provides a list of the most commonly used, namely these 13:

- boolean delete()
- boolean exists()
- String getAbsolutePath()
- String getName()
- String getParent()
- boolean isDirectory()
- boolean isFile()
- long lastModified()
- long length()
- File[] listFiles()
- boolean mkdir()
- boolean mkdirs()
- boolean renameTo(File dest)

There are 29 more, some of them with multiple overrides. createNewFile(), list(), toURI(), toURL() are some of them. There are also methods that change attributes, such as setReadOnly(), setWritable() and setLastModified(). 


