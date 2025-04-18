## The Files class

1Z0-819 wants you to be good with files and filesystems. You can expect questions about three relevant classes, namely File, Path and Files. Files is the most extensive of the three, it provides a lot of methods. Some of them, such as find(..) and walk(..) return streams, which allows you to do efficient processing with lambda's. There are methods to create BufferedReader and BufferedWriter objects more easily than via IO methods. Text files can be read at onces instead of line by line. And for copying and moving directories and files advanced methods exist that are not available in IO.

In general, Files does everything that File does, plus a lot more. You can do with less lines of code and the method names are easier to understand (mkdir vs createDirectory).

Last thing: the Files class relies strongly on the Path class in the sense that Path is very often required as argument or returned as return value.

### Methods of the Files class

Addition 2025-04-18: Here a listing, more details on each method further on:

General
- Path _createDirectory(Path dir, FileAttribute<?>... attrs)_ throws IOException
- Path _createDirectories(Path dir, FileAttribute<?>... attrs)_ throws IOException
- Path _copy(Path source, Path target, CopyOption... options)_ throws IOException
- Path _move(Path source, Path target, CopyOption... options)_
- void _delete(Path p) and boolean deleteIfExists(Path p)_
- BufferedReader _newBufferedReader(Path p)_ 
- BufferedWriter _newBufferedWriter(Path p)_
- List<String> _readAllLines(Path p)_

Returns boolean
- boolean exists(Path p, LinkOption... options)
- boolean isSameFile(Path p1, Path p2)
- boolean isDirectory(Path path, LinkOption... options)
- boolean isSymbolicLink(Path path)
- boolean isRegularFile(Path path, LinkOption... options)
- boolean isHidden(Path path)
- boolean isReadable(Path path)
- boolean isWritable(Path path)
- boolean isExecutable(Path path)

File attrubutes and properties
- long size(Path path)
- FileTime getLastModifiedTime(Path p, LinkOption... options)
- \<A extends BasicFileAttributes\> A readAttributes(Path p)
- \<V extends FileAttributeView\> V getFileAttributeView(Path p, Class\<V\> type, LinkOption... options)

Returns Stream
- Stream\<Path\> list(Path dir)
- Stream\<Path\> walk(Path p, int maxDepth, FileVisitOption... options)
- Stream\<Path\> find(Path p, int maxDepth, BiPredicate\<Path, BasicFileAttributes\> matcher, FileVisitOption... options)
- Stream\<String\> lines(Path path)

#### boolean exists(Path p, LinkOption... options)

Returns true or false. Symbolic links are followed unless you provide NOFOLLOW_LINKS. There is also a method called notExists(..) which does the opposite.

#### boolean isSameFile(Path p1, Path p2)

Checks if both paths locate the same file. Check if the files/directories exist, unless the two paths provided are identical. The method follows symbolic links. If path1 gives a symbolic link and path2 the real path to the same directory/file, true is returned. The method does noet compare the contents of files, so two copies of the same file on different locations returns false.

#### Path createDirectory(Path dir, FileAttribute<?>... attrs) throws IOException

Creates directory and throws exception if either the directory already exists or if the path is invalid. You can add FileAttributes that will apply to the new directory.

#### Path createDirectories(Path dir, FileAttribute<?>... attrs) throws IOException

Same as previous but will also create all the directories and all the required parent directories. No exception is thrown if the directory already exists.

#### Path copy(Path source, Path target, CopyOption... options) throws IOException

Performs a shallow copy. Throws exception if the target already exists, unless you add REPLACE_EXISTING as option. The other two options are COPY_ATTRIBUTES and NOFOLLOW_LINKS.

Note: the book says an IOException is being thrown, but more specifically it is an FileAlreadyExistsException that is thrown, which is a subclass of FileSystemException, which is indeed a subclass of IOException. You will mostly add 'throws IOException' to the method signature, this is what the compiler proposes.

There are overloaded variants in which either the source or target is replaced by InputStream/OutputStream. These are helpful when writing/reading to/from disk. You can choose a variety of stream types, for example System.out to print to the console. The example below works with a self-created InputStream:

```
Path source = Path.of("../text.txt");
Path dest = Path.of("../textnew.txt");

try(FileInputStream myStream = new FileInputStream(source.toString())){
    
    Files.copy(myStream, dest, REPLACE_EXISTING);
};
```

Note: toString() can be replaced with toFile() as FileInputStream has a constructor for it (but not for Path). The try-with-resources is the very right thing to do here, it ensures that the FileInputStream will be closed. REPLACE_EXISTING can be omitted if the file doesn't exist yet.

Below is an example of copying to an OutPutStream (System.out). You can use this if you want to write file content to the terminal.

```
Path source = Path.of("../text.txt");
Files.copy(source, System.out); -- contents of textfile printed to terminal
```

#### Path move(Path source, Path target, CopyOption... options)

The move method cannot only be used for moving but also for renaming. The possible options for the third argument are REPLACE_EXISTING and ATOMIC_MOVE. Throws an eexception when source doesn't exist or when the target already exists and REPLACE_EXISTING is not provided as argument. 

If source is a non-empty directory, the operation simply renames the source folder to the name of the target folder. If the target folder already contains folders or files, a DirectoryNotEmptyException is thrown. If REPLACE_EXISTING is set, source is empty and target has content, the name of target is set to the name of source. Actually a rename operation.

If source is a folder and target a file, the source folder will be renamed and have the name of the target file, including extension. Remember that Java doesn't care much about the difference between file and directory. The target file is actually lost then, its name is preserved but it has to live on as a folder.

If source is a file and target a folder (REPLACE_EXISTING is set), then the source file gets the name of the target folder. The resulting 'thing' is a file without an extension (it has the folder name of target). Nevertheless, this file without extension has the contents of the source file, or actually it is the source file.

All in all you can get strange results, so it is important that you move a file to a file or a directory to a directory. The whole move thing is actually more a rename thing.

About ATOMIC_MOVE: setting this option guarantees that the process of moving will not stop halfway for whatever reason. Either the file/directory is fully moved, or not at all. This brings some safety to the process. If the filesystem doesn't support this sort of operation, an AtomicMoveNotSupportedException is thrown. While ATOMIC_MOVE is also available for File.copy(), it will likely throw an exception there.

#### void delete(Path p) and boolean deleteIfExists(Path p)

You cannot delete a folder if it has contents. If you delete a symbolic link, only the symbolic link will be deleted, not the path that it points to. Delete throws an exception if path doesn't exist, deleteIfExists() returns false.

#### newBufferedReader(Path p) and newBufferedWriter(Path p)

Both return what their name suggests and throw IOException. This is a shorter way of creating a BufferedReader or a BufferedWriter than the way it is done with the IO class. Compare:

```
var myReader = Files.newBufferedReader(somePath);

var myReader = new BufferedReader(new FileReader(somePath));
```

As you see, the FileReader thing can be omitted and you get the same. Btw there are even easier ways to read or write files, will follow shortly.

#### List<String> readAllLines(Path p)

This is a very convenient way to read all the lines of a textfile into a List variable. No BufferedReader object is required.

### Managing file attributes

#### 'is' methods

The Files class has a set of methods to get and set file attributes. The following methods all take Path as argument (and evetually one or more LinkOptions) and return a boolean:

- isDirectory()
- isSymbolicLink()
- isRegularFile()
- isHidden()
- isReadable()
- isWritable()
- isExecutable()

#### long size(Path path)

Is meant to be used on files, not directories. When used on directory the outcome is sort of meaningless. The returned size is in bytes but might var various reasons not be exactly the same as the actual size. A NoSuchFileException is thrown if the path does not represent an existing file or folder. While you should not use it for a folder, it won't throw an exception if you do.

#### FileTime getLastModifiedTime(Path p, LinkOption... options)

Returns a timestamp which can be converted to milliseconds since January 1, 1970 using toMillis(). This 1970 thing is called the epoch time.

#### \<A extends BasicFileAttributes\> A readAttributes(Path p)

This method lets you get all fileattributes in a single call. I do not understand the structure of the interfaces, views etc, but this works:

```
Path path = Path.of("../spaghetti/pom.xml");
BasicFileAttributes b = Files.readAttributes(path, BasicFileAttributes.class);

System.out.format("creation time: %s\n", b.creationTime());
System.out.format("last modified: %s\n", b.lastModifiedTime());
System.out.format("last access: %s\n", b.lastAccessTime());
System.out.format("size: %s\n", b.size());
System.out.format("filekey: %s\n", b.fileKey());
System.out.format("is directory: %s\n", b.isDirectory());
System.out.format("is regular file: %s\n", b.isRegularFile());
System.out.format("is symbolic link: %s\n", b.isSymbolicLink());
System.out.format("is other: %s\n", b.isOther());

-- output:
-- creation time: 2025-01-29T13:50:52.0190386Z
-- last modified: 2025-02-01T20:57:23.1296665Z
-- last access: 2025-02-01T20:59:46.066098Z
-- size: 2304
-- filekey: null
-- is directory: false
-- is regular file: true
-- is symbolic link: false
-- is other: false
```

#### \<V extends FileAttributeView\> V getFileAttributeView(Path p, Class\<V\> type, LinkOption... options)

This one is the counterpart of the previos one, readAttributes, but allows you to change the value of the attributes for as far as the filesystem allows you to. lastModifiedTime for example can be set to a new value.

### Applying functional programming

Some methods in Files return a stream, which means you can chain methods like filter and map, and use functional interfaces and lambda expressions.

#### Stream\<Path\> list(Path dir)

The list method of Files is similar to File.listFiles(), except that it returns a Stream<Path> instead of File[]. It goes only one level deep. Files.list() can be used to traverse the contents of a folder and, by doing recursive stuff, make a deep copy of it instead of a shallow (as Files.copy() does). The book shows a smart custom recursive method that does that.

#### Stream\<Path\> walk(Path p, int maxDepth, FileVisitOption... options)

I used this method already, it builds a stream of all folders and files under the Path p location. You can set a maximum depth, the default is huge (max integer size). 

By default walk does not follow symbolic links. If you want it to, you can set it as option. Beware of symbolic links, as they can point to some directory outside of the scope you had in mind or because they create a circular pattern that results in a FileSystemLoopException. This exception is thrown if a path is visited twice.

#### Stream\<Path\> find(Path p, int maxDepth, BiPredicate\<Path, BasicFileAttributes\> matcher, FileVisitOption... options)

Files.find() is similar to Files.walk() but can apply a filter during the walk. This is what the BiPredicate is for. The BiPredicate has two arguments, the path and the attributes, so you can filter on both. Btw it throws IO exception. Example as in book:

```
try(var s = Files.find(path, 10, (p,a)->a.isRegularFile() &&
	p.toString().endsWith(".java")) {

s.forEach(System.out::println);
}
```

#### Stream\<String\> lines(Path path)

Similar to readAllLines() but returns a Stream\<String\> instead of a List\<String\>. The downside of storing all lines in a list is that memory consumption can get high as everything needs to be stored at once. Files.lines() doesn't have this problem, it processes lazily and doesn't need much memory.

Note that while you can use .forEach on both List\<String\>  and Stream\<String\>, you cannot make stream chains with filters, maps, distinct etc on the return value of readAllLines().


































