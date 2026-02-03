# Getting the files to compile

In my experiments in using javac (parse, analyze) I have been relying on .java files that were not really java files. I created String variables representing the contents of simple Java files and converted them to something the compiler could work with, relying on some translation method ChatGPT provided. 

As I just stumbled upon [this article](https://dzone.com/articles/jsr-199-compiler-api), I want to write a bit here on two dedicated interfaces, namely [JavaFileObject](https://docs.oracle.com/en/java/javase/21/docs/api/java.compiler/javax/tools/JavaFileObject.html) and [JavaFileManager](https://docs.oracle.com/en/java/javase/21/docs/api/java.compiler/javax/tools/JavaFileManager.html). Both live in package [javax.tools](https://docs.oracle.com/en/java/javase/21/docs/api/java.compiler/javax/tools/package-summary.html) with other relevant classes and interfaces (more interfaces than classes actually). 

The javax.tools package is part of the [java.compiler](https://docs.oracle.com/en/java/javase/21/docs/api/java.compiler/module-summary.html) module. It is to be noticed that two of its exported packages, namely `com.sun.source.tree` and `com.sun.source.util`, are not listed. I asked ChatGPT about it and it told that those two are listed in the jdk.compiler module, even if they are not located there.

Generally, the java.compiler module contains the accessible things, while the jdk.compiler module contains non-exported implementation stuff. The two modules mentioned have a sort of inbetween status, them being listed in the less accessible module (from which they are exported) is _because they are not part of what is called the 'standard API surface'._ At least that is what ChatGPT tells, it sounds reasonable.

## JavaFileObject and SimpleJavaFileObject

JavaFileObject is an interface with four abstract methods and an inner enum named 'Kind'. This is its contents, stripped from comments:

```
public interface JavaFileObject extends FileObject {

    enum Kind {
        SOURCE(".java"),
        CLASS(".class"),
        HTML(".html"),
        OTHER("");
	
        public final String extension;
        Kind(String extension) {
            this.extension = Objects.requireNonNull(extension);
        }
    }

    Kind getKind();
    boolean isNameCompatible(String simpleName, Kind kind);
    NestingKind getNestingKind();
    Modifier getAccessLevel();
)
```

JavaFileObject extends interface FileObject, that also lives in package javax.tools. This is the compact version of FileObject:

```
public interface FileObject {

    URI toUri();
    String getName();
    InputStream openInputStream() throws IOException;
    OutputStream openOutputStream() throws IOException;
    Reader openReader(boolean ignoreEncodingErrors) throws IOException;
    CharSequence getCharContent(boolean ignoreEncodingErrors) throws IOException;
    Writer openWriter() throws IOException;
    long getLastModified();
    boolean delete();
}
```

None of these two interfaces provides any implementation, but then we have SimpleJavaFileObject, which lives in javax.tools as well:

```
public class SimpleJavaFileObject implements JavaFileObject {

    protected final URI uri;
    protected final Kind kind;

    protected SimpleJavaFileObject(URI uri, Kind kind) {
        Objects.requireNonNull(uri);
        Objects.requireNonNull(kind);
        if (uri.getPath() == null)
            throw new IllegalArgumentException("URI must have a path: " + uri);
        this.uri = uri;
        this.kind = kind;
    }

    @Override
    public URI toUri() {
        return uri;
    }

    @Override
    public String getName() {
        return toUri().getPath();
    }

    @Override
    public InputStream openInputStream() throws IOException {
        throw new UnsupportedOperationException();
    }

    @Override
    public OutputStream openOutputStream() throws IOException {
        throw new UnsupportedOperationException();
    }

    @Override
    public Reader openReader(boolean ignoreEncodingErrors) throws IOException {
        CharSequence charContent = getCharContent(ignoreEncodingErrors);
        if (charContent == null)
            throw new UnsupportedOperationException();
        if (charContent instanceof CharBuffer buffer && buffer.hasArray()) {
            return new CharArrayReader(buffer.array());
        }
        return new StringReader(charContent.toString());
    }

    @Override
    public CharSequence getCharContent(boolean ignoreEncodingErrors) throws IOException {
        throw new UnsupportedOperationException();
    }

    @Override
    public Writer openWriter() throws IOException {
        return new OutputStreamWriter(openOutputStream());
    }

    @Override
    public long getLastModified() {
        return 0L;
    }

    @Override
    public boolean delete() {
        return false;
    }

    @Override
    public Kind getKind() {
        return kind;
    }

    @Override
    public boolean isNameCompatible(String simpleName, Kind kind) {
        String baseName = simpleName + kind.extension;
        return kind.equals(getKind())
            && (baseName.equals(toUri().getPath())
                || toUri().getPath().endsWith("/" + baseName));
    }

    @Override
    public NestingKind getNestingKind() { return null; }

    @Override
    public Modifier getAccessLevel()  { return null; }

    @Override
    public String toString() {
        return getClass().getName() + "[" + toUri() + "]";
    }
}
```

The relevant part is the protected constructer, that allows for extending this class. The two instance variables 'uri' and 'kind' are set by the arguments of the constructor and cannot be null. The URI variable must have a path. Both uri and path can be accessed with a getter. Most other methods do not have a workable implementation, except for getName, openReader, openWriter and toString. 

### What the JavaCompiler needs

The JavaCompiler interface, also in javax.tools, has a method that returns a CompilationTask object. This method is needed if you want to compile things, we have encountered it in the previous blog post. This is its source code:

```
    CompilationTask getTask(Writer out,
                            JavaFileManager fileManager,
                            DiagnosticListener<? super JavaFileObject> diagnosticListener,
                            Iterable<String> options,
                            Iterable<String> classes,
                            Iterable<? extends JavaFileObject> compilationUnits);
```

As you see, the last argument is `Iterable<? extends JavaFileObject>` which means that we either need to create JavaFileObjects from our .java or .class files, or files that descend from JavaFileObject. To create these objects, we need an implementation of it, and that is what we use SimpleJavaFileObject for. 

### Using SimpleJavaFileObject

SimpleJavaFileObject is designed in such a way that when you implement it, you will create a type that contains exactly the fields and methods that are required to have it work well as input to the getTask(..) method. Once you extend SimpleJavaFileObject, you need to provide a constructor that calls the parent constructor (via super). This is because SimpleJavaFileObject lacks a no-argument constructor. 

Furthermore you need to implement one method, which is really being used by the getTask method. It is this one:

```
CharSequence getCharContent(boolean ignoreEncodingErrors) throws IOException;
```

SimpleJavaFileObject has an implementation of this method but it does only throw an error, which means code will fail at runtime:

```
    @Override
    public CharSequence getCharContent(boolean ignoreEncodingErrors) throws IOException {
        throw new UnsupportedOperationException();
    }
```

The implementation I used in my experiments looks like this:

```
public class StringTypeJavaFileObject extends SimpleJavaFileObject {

    // The parent object has two private instance variables, namely URI and Kind

    private final String code;

    public StringTypeJavaFileObject(String className, String code) {
        super(URI.create("string:///" + className.replace('.', '/') +
                        JavaFileObject.Kind.SOURCE.extension),
                Kind.SOURCE);
        this.code = code;
    }

    @Override
    public CharSequence getCharContent(boolean ignoreEncodingErrors) {
        return code;
    }
}
```

Here you see that, to make the constructor work, the two-argument parent constructor is being called and it sets the inherited instance variables `protected final URI uri` and `protected final Kind kind`. An extra instance field `private final String code` is added and used as return value for the getCharContent(..) method. 

Note that it is very similar to [this](https://dzone.com/articles/jsr-199-compiler-api).

#### `URI.create("string:///" + className.replace('.', '/')`

I had question about this first argument in the super constructor. ChatGPT told me that the generated URI is not being used as a real location but as a sort of valid name. The first part, `string://`, has no specific meaning in URI-land and the last part is a path that in this case does not have to point to something real.

### Why JavaFileObject is so extensive

ChatGPT explained that JavaFilObject, with all abstract methods inherited from FileObject, is a class that serves in multiple roles, thus not only in the role I use it for in this context. Depending on the role you need to provide implementations for other methods, methods that now simply return exceptions (UnsupportedOperationException) or values that make no sense (0L in getLastModified()). 

For now, we simply must create JavaFileObjects that can provide the values of Kind kind, URI uri and CharSequence filecontent. Everything else can be done without.

## JavaFileManager

If we get back to this method from JavaCompiler:

```
    CompilationTask getTask(Writer out,
                            JavaFileManager fileManager,
                            DiagnosticListener<? super JavaFileObject> diagnosticListener,
                            Iterable<String> options,
                            Iterable<String> classes,
                            Iterable<? extends JavaFileObject> compilationUnits);
```

We see that the first argument is of interface type JavaFileManager. You can provide null as argument, in that case a default under-the-hood implementation will be used, namely JavacFileManager. This class lives in package `com.sun.tools.javac.file` and is not exported. It has a ton of methods.

In my experiment I have set the value to null, meaning I do not have any reference variable pointing to the JavaFileManager object in use. This means I cannot:

- compile without touching disk
- collect class files in memory
- control module resolution
- isolate compilation from the system classpath
- support non-file-backed sources and outputs

This is not a problem for now but for more advanced use it is. The Java(c)FileManager resolves all th ings related to the classpath, the modulepath, the files, and who can see what filewise. It is a deep topic and I will dive deeper in it in the future. For now this is it.

