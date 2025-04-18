## Files class, file attributes

In an earlier post I covered the Files class with its methods but upon revisiting it I noticed that not everything was detailed out enough. The Files class has lots of methods and one of the things I need to understand better is how Java deals with file attributes.

### The BasicFileAttributes interface

This is an interface in the java.nio.file.attribute package. It's documentation says: _Basic file attributes are attributes that are common to many file systems and consist of mandatory and optional file attributes as defined by this interface._ This interface is extended by interfaces DosFileAttributes and PosixFileAttributes. DosFileAttributes is implemented by class WindowFileAttributes, PosixFileAttributes by classes related to Unix-based systems. 

BasicFileAttributes defines nine abstract methods, namely:
- FileTime **creationTime**()
- Object **fileKey**()
- boolean **isDirectory**()
- boolean **isOther**()
- boolean **isRegularFile**()
- boolean **isSymbolicLink**()
- FileTime **lastAccessTime**()
- FileTime **lastModifiedTime**()
- long **size**()

As far as Windows is concerned, four more abstract methods are defined in DosFileAttributes:
- boolean **isReadOnly**()
- boolean **isHidden**()
- boolean **isArchive**()
- boolean **isSystem**()

To make it complete, the PosixFileAttributes interface has three extra abstract methods:
- Userprincipal **owner**()
- GroupPrincipal **group**()
- Set\<PosixFilePermission\> **permissions**()

Remarkable: all these methods are no-argument.

BasicFileAttributes and its subinterfaces are the return value of the following method:

 _public static \<A extends BasicFileAttributes\> A **readAttributes**(Path p, Class\<A\> type, LinkOption... options) throws IOException_

You can apply it in several ways:

```
Path path  = Paths.get("C:/Kuips files/Java projects/generics/text.txt");
BasicFileAttributes basic = Files.readAttributes(path, BasicFileAttributes.class);
DosFileAttributes dos = Files.readAttributes(path, DosFileAttributes.class);

System.out.println(basic.isSystem());  // compile error - isSystem not in BasicFileAttributes interface
System.out.println(dos.isSystem());    // no compile error
```

Btw the LinkOption... options parameter relates to enum LinkOption which contains just one value, NOFOLLOW_LINKS. Adding this as argument means that symbolic links will not be followed.

### BasicFileAttributeView

The BasicFileAttributeView is another interface in the java.nio.file.attribute package. It is the return value of the following method:

_public static <V extends FileAttributeView> V **getFileAttributeView​**(Path path, Class<V> type, LinkOption... options)_

This declaration is very similar to that of readAttributes method. BasicFileAttributeView has subinterfaces similar to those of BasicFileAttribute, namely DosFileAttributeView and PosixFileAttributeView. 

The methods for BasicFileAttributeView are: 

- String **name**()
- BasicFileAttributes **readAttributes**()
- void **setTimes**​(FileTime lastModifiedTime, FileTime lastAccessTime, FileTime createTime)

For DosFileAttributeView: 

- DosFileAttributes **readAttributes**()
- void **setReadOnly**(boolean value)
- void **setHidden**​(boolean value)
- void **setSystem**​(boolean value)
- void **setArchive**​(boolean value)

For PosixFileAttributeView:

- PosixFileAttributes **readAttributes**()
- void **setPermissions​**(Set<PosixFilePermission> perms)
- void **setGroup**​(GroupPrincipal group)

Using this methods can be done with several interfaces:

```
Path path  = Paths.get("C:/Kuips files/Java projects/generics/text.txt");

BasicFileAttributeView basicView = Files.getFileAttributeView(path, BasicFileAttributeView.class);
BasicFileAttributes attributes = basicView.readAttributes();

long oldTime = attributes.lastModifiedTime().toMillis();
long newTime = oldTime + 1_000_000;
FileTime newFileTime = FileTime.fromMillis(newTime);

basicView.setTimes(newFileTime, null, null);
```

```
Path path  = Paths.get("C:/Kuips files/Java projects/generics/text.txt");

DosFileAttributeView dosView = Files.getFileAttributeView(path, DosFileAttributeView.class);
DosFileAttributes attributes = dosView.readAttributes();

long oldTime = attributes.lastModifiedTime().toMillis();
long newTime = oldTime + 1_000_000;
FileTime newFileTime = FileTime.fromMillis(newTime);

dosView.setTimes(newFileTime, null, null);
```

I was wondering why so much complexity was involved, knowing that the Files class has a setAttribute(..) method that basically does the same thing. And File has a setLastModified(long time) as well. I asked ChatGPT (sorry Perplexity) and its answer had the following points:

- Files.setAttribute is a higher level, more general method with more overhead, less efficient.
- Repeated access (batches) is faster with getFileAttributeView().setTimes().
- Files.setAttribute is more error prone as it uses string-based names for attributes, won't be checked at compile time.
- While BasicAttributeView only lets you modify FileTime values of created, modified and last access, DosFileAttributeView does some more, also things that cannot be done easily in any other way (I have to check this).

Anyhow, I know a bit more about _Files.readAttributes_, that returns the \<A extends BasicFileAttributes\> object, and _Files.getFileAttributeView_, that returns the \<A extends FileAttributeView\> object. The latter is a bit contrived but I suppose there are situations in which you have to deal with large batches where outcome and processing speed is more important than minimal coding effort.



