## The Path interface

The exam involves both the IO and the NIO.2 package. The former has the File class, the latter has the Path interface and the Files class. Here I will summarize the Path interface, using [the book](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1) chapter 20 plus the source files and the Oracle documentation.

Note: there is no package named nio.2, there is only java.io and java.nio. NIO.2 is the name given to the 2007 updates/additions and the files belonging to this release can be found in the 'file' package that is in the 'nio' package. 


### Symbolic links

Path class can deal with symbolic links while File class can't. 'A symbolic link is a special file within a file system that serves as a reference pointer to another file or directory.' Think of a Favorites folder in your windows system.

### Creating a Path object

As Path is an interface, you cannot instantiate it by using "new Path()". But there are other methods, namely the following:

#### 1 - Use its static factory method 'of', like this:

```
Path.of("C:\\users\\user\\myProject\\myText.txt");
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

"WARNING: This interface is only intended to be implemented by those developing custom file system implementations. Methods may be added to this interface in future releases."







