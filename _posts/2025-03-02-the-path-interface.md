## The Path interface

The exam involves both the IO and the NIO.2 package. The former has the File class, the latter has the Path interface and the Files class. Here I will summarize the Path interface, using [the book](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1) chapter 20 plus the source files and the Oracle documentation.

Note: there is no package named nio.2, there is only java.io and java.nio. NIO.2 is the name given to the 2007 updates/additions and the files belonging to this release can be found in the 'file' package that is in the 'nio' package. 


### Symbolic links

Path class can deal with symbolic links while File class can't. 'A symbolic link is a special file within a file system that serves as a reference pointer to another file or directory.' Think of a Favorites folder in your windows system.

### Creating a Path object

As Path is an interface, you cannot instantiate it by using "new Path()". But there are other methods, namely the following:

#### 1 - Use its static factory method 'of', like this:

```
Path.of("C:\users\user\myProject\myText.txt");
```

You can pass multiple String arguments to the constructor, it is of the varargs type and they will be concatenated.

#### 2 - Use the Paths class 
The Paths class (with an s) is from the NIO.2 release. It has a static constructor very similar to the previous one:

```
Paths.get("C:\", "MyFiles", "project 1");
```

Note that I supplied multiple arguments, they will be concatenated just as in the previous method.

#### 3 - Convert URI object to Path object

You can create an URI object (from the java.net library) and supply it as an argument to Path.of:

```
URI myURI = new URI("file://C:...");

Path myPath = Path.of(myURI);
or
Path myPath = Paths.get(myURI);
```
note: I'm not familiar with URI's.

#### 4 - Use the FileSystems and FileSystem class

FileSystems and FileSystem are both classes in the NIO.2 release, they originate from Java 1.7 (or Java 7). It is possible to create a Path object by doing the following:

```
FileSystem fs = FileSystems.getDefault(); --create FileSystem object
Path myPath = fs.getPath("projects/Havana/src/some.java");
```

You can do it in one line:

```
Path myPath = FileSystems.getDefault().getPath("projects/Havana/src/some.java");
```

#### 5 - Use the good old File class

File has a toPath() method, added in the Java 7 release together with NIO.2. 

```
File myFile = newFile("images/photo1.jpg");
Path myPath = myFile.toPath();
```

### NIO.2 is better at connecting to remote file system

NIO.2 is better than IO in connecting to a remote filesystem, which is a major advantage. For this the FileSystem class was introduced. Generally Path is used for paths within the system, FileSystem for remote paths.

### Path methods

The book doesn't delve into the static methods of Path and actually there is just one, namely Path.of(String(s) or URI). There is a list of abstract methods and default methods in the Path body and actually I don't know what they are used for, as Path is not implemented by other classes. Maybe this text from the Oracle documentation helps us out?

*"WARNING:* This interface is only intended to be implemented by those developing custom file system implementations. Methods may be added to this interface in future releases."

*WAIT:* I now see that the abstract methods are actually implemented, namely in the JDK, for example in the file WindowsPath.java. I suppose this means that if you create a path object, on a windows machine you actually create a WindowsPath object, and all the interface methods of Path will really work.

Okay, now that this is out of the way, an overview of the methods that are named in the book:

- Path of(String, String...)
- URI toURI()
- File toFile()
- String toString()
- int getNameCount()
- Path getName(int index)
- Path subpath(int beginindex, int endindex)
- Path getFileName()
- Path getParent()
- Path getRoot()
- boolean isAbsolute()
- Path toAbsolutePath()
- Path relativize(Path other)
- Path resolve(Path other)
- Path normalize()
- Path toRealPath(LinkOption...)

The first one is a static factory constructor, 2 and 3 are methods to convert a Path to a different object type (URI and File). The toString method returns the path string, but if you want to print it via System.out.println you can skip .toString because it will automatically do so.

Path.getNameCount() returns the number of directories/files in the path, excluding the root. "C:/My Files/Project Java" as argument returns 2 as "C:", being the root, is not counted. In UNIX the root is "/" and that one is not counted either. Path.getName(int index) returns the file/directory with the corresponding index. Example:

```
Path p = Path.of("C:/My Files/Project Java/main/java.txt");

p.getNameCount(); 
-- returns 4

System.out.println(p.getName(0));
-- prints "My Files"
System.out.println(p.getName(3));
-- prints "java.txt"
```

Path.subpath(int beginindex, int endindex) returns a Path object containing a subpath. It throws an IllegalArgumentException if the index is out of bounds:

```
Path p = Path.of("C:/My Files/Project Java/main/java.txt").subPath(1,3);
-- returns "Project Java/main"

Path p = Path.of("C:/My Files/Project Java/main/java.txt").subPath(1,4);
-- IllegalArgumentException
```

Path.getFileName() returns the last part of the path, it can be a file or a directory. Note that these methods are not so interested in the difference between file and directory. If there is only a root in the path, the return value is null. Path.getParent() and Path.getRoot() are similar, returning null if they cannot answer differently. Path.getRoot() returns null if the path is not absolute.

Path.isAbsolute() returns true if the path is absolute, given the operating system. For WIndows the path must start with "C:/" or something similar, in UNIX "/". Path.toAbsolutePath() makes an absolute path out of a relative path. The method assumes that the relative path provided starts at the current directory (or actually one level deeper).

Path.relativize() returns a path that indicates how to get from path 1 to path 2. The return path is the thing you would have to type after cd in the terminal assuming that your current directory is path 1 and you want to set your current directory to path 2. Example:

```
Path mp1 = Path.of("C:/Kuips files/Java projects");
Path mp2 = Paths.get("C:/Games/replays");

System.out.println(mp1.relativize(mp2));

-- returns "..\..\Games\replays" 
```

A caveat here is that both paths need either be absolute or relative. Mixing those results in IllegalArgumentException. If both are absolute and you work on Windows, they need to have the same drive letter. If not, also an IllegalArgumentException.

The resolve(Path other) method concatenates two paths:

```
Path p1 = Path.of("new folder 2");
Path p2 = Path.of("text.txt");

Path newpath = p1.resolve(p2);
-- returns "new folder 2\text.txt"
```

If the argument (p2) an absolute path while the other (p1) is not, resolve(..) will return p2 as this is sort of the only possible answer. If p1 is an absolute path and p2 isn't, they will be neatly concatenated. If both are absolute, p2 will be returned:

```
Path p1 = Path.of("new folder 2").toAbsolutePath();
Path p2 = Path.of("text.txt").toAbsolutePath();

Path newpath = p1.resolve(p2);
-- returns "C:\Kuips files\Java projects\generics\text.txt"
```

It works best if p1 is absolute and p2 isn't, or if both are relative.

The normalize() method turns paths with . and/or .. in it into regular paths without them. .. refers to the parent directory while . refers to the current directory. Example:

```
System.out.println(Path.of("./Java projects/myText.txt").normalize());
-- returns "Java projects\myText.txt"

System.out.println(Path.of("Java projects/../myText.txt").normalize());
-- returns "myText.txt"

System.out.println(Path.of("C:/./Java projects/files/../myText.txt").normalize());
-- returns "C:\Java projects\myText.txt"
```

The realPath(LinkOption...) method does a few things. It normalizes the path, it makes it an absolute path, and while you can pass symbolic links, it will convert them to their real substitute. Path.realPath(LinkOption...) differs from most other methods in that it is really concerned with the existence of the path. If it doesn't exist, an IOException is thrown. 

If NOFOLLOW_LINKS is added as option, symbolic links will not be resolved.

























