# Class JavaCompiler

In the jdk.compiler module there is package com.sun.tools.javac.main which contains JavaCompiler. In this blog post I will discuss its contents, encouraged by ChatGPT who told me that this class gives good insight in the different phases of compilation.

Having said that, I know that I will encounter a lot of mechanisms within classes directly related to JavaCompiler. I am going to dive in these rabbit holes without reservations, hoping to learn a lot more about advanced Java programming.

By writing this post, I hope to learn more about the folwoing topics:

- The role, use and internals of related classes like Context, Enter, Symtab, Classreader
- The specifics of different phases
- How JavaCompiler relates to JavadocTool, interface JavaCompiler and JavacTaskImpl
- The reason why multiple instances of JavaCompiler are being created during compilation
- How comments and annotations are part of this

Btw JavaCompiler has all sorts of layers on top of it, you do not access it directly. And this class should not be confused with public interface JavaCompiler (javax.tools.JavaCompiler) which has com.sun.tools.javac.api.JavacTool as runtype (thus not this JavaCompiler class).

## Overall structure

JavaCompiler looks like it is well structured. Every required dependency has a proper field with proper javadoc comments, making it easier to understand what the class is doing and how it relies on other classes.

The general structure, omitting a lot, can be summarized as this:

```
field Context.Key<JavaCompiler> compilerKey

normal constructor + .instance method to retrieve JavaCompiler belonging to specific context

many fields (Symtab, Source, Flow, Lower, Gen, ModuleFinder etc)

many boolean flags (verbose, sourceOutput etc) 

method compile(...)  // this is the main method

lots of other methods and stuff
```

## Context

The first line of JavaCompiler is this:

```
public static final Context.Key<JavaCompiler> compilerKey = new Context.Key<>();
```

The first thing to know is that this line can be found in (almost) identical form in all other classes that play lead roles during compilation. Think of Symtab, ClassFinder and Enter.

The essential thing here is that Context has a field `protected final Map<Key<?>,Object> ht = new HashMap<>();`. In this map one and only one instance of all required classes is stored. To guarantee uniqueness and to be able to store instances of different types, Context uses a trick. It has a static inner class named Key with a type parameter:

```
    public static class Key<T> {
        // note: we inherit identity equality from Object.
    }
```

Every class being used for the compilation process generates a Key object as a static final field upon class initialization, like in the first line of JavaCompiler. This Key object is unique by definition, as it is a reference object (and not a String).

The constructor of each of these classes is protected and will not be called directly. To instantiate a class, an instance method must be called. This is the instance method for JavaCompiler, all other classes participating in this construction have the same instance method:

```
    public static JavaCompiler instance(Context context) {
        JavaCompiler instance = context.get(compilerKey);
        if (instance == null)
            instance = new JavaCompiler(context);
        return instance;
    }
```

This method checks if the instance is already added to Context context. If not, the constructor is being called. This constructor has this line:

```
    context.put(compilerKey, this);
```

So basically it adds itself to the map in Context where all the instances of the various classes live. The `put` method in Context is quite sophisticated:

```
    public <T> void put(Key<T> key, T data) {
        if (data instanceof Factory<?>)
            throw new AssertionError("T extends Context.Factory");
        checkState(ht);
        Object old = ht.put(key, data);
        if (old != null && !(old instanceof Factory<?>) && old != data && data != null)
            throw new AssertionError("duplicate context value");
    }
```

What is so great about this is the generics. Remember that the map in which instances are stored is ot type `Map<Key<?>,Object>`. The Key instance has a generic parameter that must match the type of the Object. This type similarity is guaranteed because the `put` method enforces type equality. Key must have T as type, and Object must be of type T.

This compile type safety is just really clever. In the end we have a context, stored as a Map of key value pairs, with one guaranteed unique instance of every object type that is required for the compilation process.

And if we need that object, we just use Context's get method:

```
    public <T> T get(Key<T> key) {
        checkState(ht);
        Object o = ht.get(key);
        if (o instanceof Factory<?> fac) {
            o = fac.make(this);
            if (o instanceof Factory<?>)
                throw new AssertionError("T extends Context.Factory");
            Assert.check(ht.get(key) == o);
        }

        /* The following cast can't fail unless there was
         * cheating elsewhere, because of the invariant on ht.
         * Since we found a key of type Key<T>, the value must
         * be of type T.
         */
        return Context.uncheckedCast(o);
    }

    private static <T> T uncheckedCast(Object o) {
        return (T)o;   
    }
```

There are some nested methods in it but in the end it is guaranteed to work. Btw the uncheckedCast method at the end has a TODO comment that says _TODO: This method should be removed and Context should be made type safe. This can be accomplished by using class literals as type tokens._

## Contents of Context

Now that we know what Context is and how it guarantees uniqueness of its contents, it is interesting to know what is in Context. Basically you can put everything in it but when compiling there is a specific collection of types that are present. Context is injected into JavaCompiler via the constructor. Let's look  at the first part of the constructor:

```
    public JavaCompiler(Context context) {
        this.context = context;
        context.put(compilerKey, this);

        // if fileManager not already set, register the JavacFileManager to be used
        if (context.get(JavaFileManager.class) == null)
            JavacFileManager.preRegister(context);
```

As can be seen, the JavaCompiler instance itself is added to the injected Context object. The lines about JavaFileManager probably have to do with the question whether a JavaFileManager is provided as argument when creating a JavacTask. If you set that argument to null, a default implementation is added to the context here.

This probably means that the injected Context object already contains a `List<JavaFileObject>`, as this is a required argument in the getTask method of JavacTool. Let's see what JavacTool puts in context. I only copied the lines from JavacTool that added stuff to Context:

```
context.put(DiagnosticListener.class, ccw.wrap(diagnosticListener));

context.put(Log.errKey, new PrintWriter(System.err, true));

context.put(JavaFileManager.class, fileManager);
```

Interestingly Context has an overload for the put method that allows you to add an entry to the basic map without having to create a Key object yourself. Anyhow, what is still missing is the `List<JavaFileObject>`. This list is finding its way into Context via the Arguments object. 

This is code in JavacTool:

```
            Arguments args = Arguments.instance(context);  // adding args to Context
            args.init("javac", options, classes, compilationUnits);
```

This is the init method in Arguments:

```
    public void init(String ownName,
            Iterable<String> options,
            Iterable<String> classNames,
            Iterable<? extends JavaFileObject> files) {
        this.ownName = ownName;
        this.classNames = toSet(classNames);
        this.fileObjects = toSet(files);
        this.files = null;
        errorMode = ErrorMode.ILLEGAL_ARGUMENT;
        if (options != null) {
            processArgs(toList(options), Option.getJavacToolOptions(), apiHelper, false, true);
        }
        errorMode = ErrorMode.ILLEGAL_STATE;
    }
```

The list of JavaFileObjects is converted to a set, guaranteeing uniqueness, and stored as field in the Arguments object. To get it, you need to call this method in Arguments:

```
    public Set<JavaFileObject> getFileObjects() {
        if (fileObjects == null) {
            fileObjects = new LinkedHashSet<>();
        }
        if (files != null) {
            JavacFileManager jfm = (JavacFileManager) getFileManager();
            for (JavaFileObject fo: jfm.getJavaFileObjectsFromPaths(files))
                fileObjects.add(fo);
        }
        return fileObjects;
    }
```

So up until now we have DiagnosticListener, Log, JavaFileManager and Arguments. Additional type instances are added in the constructor of JavaCompiler:

```
        names = Names.instance(context);
        log = Log.instance(context);
        diagFactory = JCDiagnostic.Factory.instance(context);
        finder = ClassFinder.instance(context);
        reader = ClassReader.instance(context);
        make = TreeMaker.instance(context);
        writer = ClassWriter.instance(context);
        jniWriter = JNIWriter.instance(context);
        enter = Enter.instance(context);
        todo = Todo.instance(context);

        fileManager = context.get(JavaFileManager.class);  // already in context
        parserFactory = ParserFactory.instance(context);
        compileStates = CompileStates.instance(context);

        try {
            // catch completion problems with predefineds
            syms = Symtab.instance(context);
        } catch (CompletionFailure ex) {
            // inlined Check.completionError as it is not initialized yet
            log.error(Errors.CantAccess(ex.sym, ex.getDetailValue()));
        }
        source = Source.instance(context);
        preview = Preview.instance(context);
        attr = Attr.instance(context);
        analyzer = Analyzer.instance(context);
        chk = Check.instance(context);
        gen = Gen.instance(context);
        flow = Flow.instance(context);
        transTypes = TransTypes.instance(context);
        lower = Lower.instance(context);
        annotate = Annotate.instance(context);
        types = Types.instance(context);
        taskListener = MultiTaskListener.instance(context);
        modules = Modules.instance(context);
        moduleFinder = ModuleFinder.instance(context);
        diags = Factory.instance(context);
        dcfh = DeferredCompletionFailureHandler.instance(context);

        finder.sourceCompleter = sourceCompleter;  // no addition to context
        modules.findPackageInFile = this::findPackageInFile;  // no addition to context
        moduleFinder.moduleNameFromSourceReader = this::readModuleName;  // no addition to context

        options = Options.instance(context);
```

Everywhere you see the static instance(context) method being used, a class instance is added to the context. I count 30, plus the 4 items added earlier makes 34.

## The compile function

JavaCompiler is the central class for compiling but there is a difference between compiling from the command line and compiling fromout an application, using task.parse() and task.analyze(). 

### Compiling with command line

In the first case all compilation is done at once by the compile method, but it starts at `com.sun.tools.javac.main.Main`. This Main class has its own compile method, this method is extensive and has this signature:

```
	public Result compile(String[] argv, Context context)
```

As you see it takes command line arguments and Context as parameters. It fills the context and then you find these calls:

```
	JavaCompiler comp = JavaCompiler.instance(context);

	comp.compile(args.getFileObjects(), args.getClassNames(), null, List.nil());
```

So from there on JavaCompiler goes to work.

### Compiling fromout application

When you compile programmatically as I do, there is a central role for `com.sun.tools.javac.api.JavacTaskImpl`. ChatGPT calls JavacTaskImpl a 'controlled facade over JavaCompiler'. It allows for doing the compilation process in small steps instead of doing all at once. 

JavacTaskImpl holds a reference to JavaCompiler as it gets Context injected in its constructor (and JavaCompiler is in Context). JavaTaskImpl is now able to call the different methods in JavaCompiler that start specific phases of the compilation process. It has these methods:

```
// parsing, this method is called by other method 'parse'. 
    private Iterable<? extends CompilationUnitTree> parseInternal() {
        try {
            prepareCompiler(true);
            List<JCCompilationUnit> units = compiler.parseFiles(args.getFileObjects());
            for (JCCompilationUnit unit: units) {
                JavaFileObject file = unit.getSourceFile();
                if (notYetEntered.containsKey(file))
                    notYetEntered.put(file, unit);
            }
            return units;
        }
        finally {
            parsed = true;
            if (compiler != null && compiler.log != null)
                compiler.log.flush();
        }
    }

// Enter phase, I give signature only:
public Iterable<? extends Element> enter(Iterable<? extends CompilationUnitTree> trees)

// Analyze (as in task.analyze()
    public Iterable<? extends Element> analyze(Iterable<? extends Element> classes) {
        enter(null);  // ensure all classes have been entered

        final ListBuffer<Element> results = new ListBuffer<>();
        try {
            if (classes == null) {
                handleFlowResults(compiler.flow(compiler.attribute(compiler.todo)), results);
            } else {
                Filter f = new Filter() {
                    @Override
                    public void process(Env<AttrContext> env) {
                        handleFlowResults(compiler.flow(compiler.attribute(env)), results);
                    }
                };
                f.run(compiler.todo, classes);
            }
        } finally {
            compiler.log.flush();
        }
        return results;
    }

// Generate (bytecode I think)
    public Iterable<? extends JavaFileObject> generate(Iterable<? extends Element> classes) {
        final ListBuffer<JavaFileObject> results = new ListBuffer<>();
        try {
            analyze(null);  // ensure all classes have been parsed, entered, and analyzed

            if (classes == null) {
                compiler.generate(compiler.desugar(genList), results);
                genList.clear();
            }
            else {
                Filter f = new Filter() {
                        @Override
                        public void process(Env<AttrContext> env) {
                            compiler.generate(compiler.desugar(ListBuffer.of(env)), results);
                        }
                    };
                f.run(genList, classes);
            }
            if (genList.isEmpty()) {
                compiler.reportDeferredDiagnostics();
                cleanup();
            }
        }
        finally {
            if (compiler != null)
                compiler.log.flush();
        }
        return results;
    }
```

So basically, JavacTaskImpl is able to call the methods of JavaCompiler individually, making it a good tool for programmatic compilation. JavaCompiler is the class where the methods live that start the different phases, and these methods are either called from its own compile method or form the parse(), analyze(), enter() and generate() methods of JavacTaskImpl.

## How parsing is activated

JavaCompiler has multiple overloaded parse() methods that in the end all call this one:

```
    private JCCompilationUnit parse(JavaFileObject filename, CharSequence content, boolean silent) {
        long msec = now();
        JCCompilationUnit tree = make.TopLevel(List.nil());
        if (content != null) {
            if (verbose) {
                log.printVerbose("parsing.started", filename);
            }
            if (!taskListener.isEmpty() && !silent) {
                TaskEvent e = new TaskEvent(TaskEvent.Kind.PARSE, filename);
                taskListener.started(e);
                keepComments = true;
                genEndPos = true;
            }
            Parser parser = parserFactory.newParser(content, keepComments(), genEndPos,
                                lineDebugInfo, filename.isNameCompatible("module-info", Kind.SOURCE));
            tree = parser.parseCompilationUnit();
            if (verbose) {
                log.printVerbose("parsing.done", Long.toString(elapsed(msec)));
            }
        }

        tree.sourcefile = filename;

        if (content != null && !taskListener.isEmpty() && !silent) {
            TaskEvent e = new TaskEvent(TaskEvent.Kind.PARSE, tree);
            taskListener.finished(e);
        }

        return tree;
    }
```

The line `Parser parser = parserFactory.newParser(..)` creates a new Parser of runtime type JavacParser. Note that parserFactory is in Context, so with Context you can always create a Parser instance. The line `tree = parser.parseCompilationUnit();` gives a JCCompilationUnit object, which is the top of the AST. This is exactly what you want.









