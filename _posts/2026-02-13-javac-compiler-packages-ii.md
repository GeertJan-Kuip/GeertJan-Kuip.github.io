# Javac compiler packages 2

In the previous blog post I discussed all the packages of java.compiler and jdk.compiler. In this post I want to describe a mental model that helps me to get more familiar with the whole compiler thing.

In this post I will omit those parts that have to do with annotations and Javadoc simply because I am not familiar with it yet.

## Compilation stages

Compilation happens in phases, and each next phase requires extra tooling. I will try to connect packages to the first three phases, which gives some structure. Here we go:

### Parse

The Lexer tokenizes the source files and the parser builds the syntax tree. There are AST's but they are not enriched with symbols or types yet.

The packages that deal with this phase are:

```
- javax.tools                 interfaces of JavaCompiler, JavaFileManager, JavaFileObject
- com.sun.source.tree         all the Tree interfaces
- com.sun.source.util         JavacTask, Trees, TreeScanner, SimpleTreeVisitor
- com.sun.tools.javac.api     implementations of interfaces and abstract classes found in com.sun.source.util
- com.sun.tools.javac.file    tooling for working with files, JavacFileManager
- com.sun.tools.javac.parser  all lexer and parser stuff
- com.sun.tools.javac.tree    JCTree lives here
- com.sun.tools.javac.util    all sorts of basic objects
```

The packages not relevant in this stage are (only from java.compiler):

```
javax.lang.model.element      requires symbols
javax.lang.model.type         requires types
javax.lang.model.util         scanners and visitors working on symbols and types
```

### Enter

Symbols for top-level declarations (class, method, field) are created and added to AST. Scopes are created and populated.

Packages that make their entrance are:

```
javax.lang.model.element      interfaces for Symbol objects
javax.lang.model.util         Visitors and scanners for Element (Symbol)
com.sun.tools.javac.code      Scope, Symbol, Symtab, Flags and Kinds live here
com.sun.tools.javac.comp      Here the Enter class is found
com.sun.tools.javac.model     Contains an JavacElements utility class
```

### Attr (Attribution)

Find/create a Symbol for every name, determine expression types, overload resolution, check method calls, resolve generics, infer types and attach type objects to trees.

Packages now entering the game:

```
javax.lang.model.type          Interfaces for Type objects
```

### Further phases

There are more phases, namely:

- Flow (data flow analysis)
- TransTypes (type erasure)
- Lower (simplify language constructs)
- Desugar (further simplification)
- Generate (create bytecode)
- Write (writing class files to disk)

## Big directories

Another way of structuring can be to explicitly describe those directories that contain the most packages:

|Package|Short description|Detailed description|
|----|----|----|
|javax.lang.model|Element and TypeMirror|Interfaces for elements/symbols, types, visitors and scanners for elements and types|
|com.sun.source|Tree and DocTree|Tree and DocTree interfaces, scanners, visitors, abstract classes Trees, JavacTask|
|com.sun.tools.javac|Hardcore implementations|14 packages, everything required under the hood up until bytecode generation|
