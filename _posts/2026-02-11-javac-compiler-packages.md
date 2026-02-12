# Javac compiler packages

Working with compiler internals means working with two modules, namely platform module *java.compiler* and JDK implementation module *jdk.compiler*. The former, according to the javadoc of its module-info file, _defines the Language Model, Annotation Processing, and Java Compiler API's_. The latter _defines the implementation of the system Java compiler and its command line equivalent, javac._

In this blog post I'll discuss the contents of both modules, not exhaustive but enough to get some idea what packages are in it and how these packages can be utilized.

## The java.compiler module

The java.compiler module is completely open, meaning that every package is exported. Using them as imports in your code will not force you to compile and/or run the application with --add-exports flags.

### module-info

This is the contents of the module-info.java file. All packages are exported.

```
module java.compiler {
    exports javax.annotation.processing;
    exports javax.lang.model;
    exports javax.lang.model.element;
    exports javax.lang.model.type;
    exports javax.lang.model.util;
    exports javax.tools;

    uses javax.tools.DocumentationTool;
    uses javax.tools.JavaCompiler;
}
```

The 'uses' lines mean that the module relies on two services, these services live in another module and are implementations of the DocumentationTool and JavaCompiler interfaces. The implementation of DocumentationTool lives in the jdk.javadoc module, whose module-info file has corresponding declarations:

```
    provides javax.tools.DocumentationTool with
        jdk.javadoc.internal.api.JavadocTool;
```

The implementation of JavaCompiler lives in jdk.compiler, which has this line in its module-info:

```
    provides javax.tools.JavaCompiler with
        com.sun.tools.javac.api.JavacTool;
```

These matching declarations, `uses` and `provides,` connect the open java.compiler module to the mostly closed jdk.compiler and jdk.javadoc modules where the implementations live.

### Packages in java.compiler

The java.compiler module has only six packages, they are all listed with an exports clause in module-info. They can be recognized by the 'javax' element with which the package name starts. Easy rule of thumb: packages starting with 'javax' belong in java.compiler, all other packages belong to jdk.compiler.

Here follow short descriptions of these packages. The italic text is from the official javadoc:

#### javax.annotation.processing

_Facilities for declaring annotation processors and for allowing annotation processors to communicate with an annotation processing tool environment._

I do not go into depth into this one. Thirteen files, mostly interfaces, one abstract class, one utility class, one exception class.

#### javax.lang.model

Base package, there are three nested packages within this folder. Nothing important in it, just three files. Interface AnnotatedConstruct, parent of Element, lives here. 

#### javax.lang.model.element

_Interfaces used to model elements of the Java programming language._

Here the Element interface, which is implemented by Symbol, is found. There are mainly child interfaces of Element like ExecutableElement, ModuleElement and Parameterizable in it. 

Furthermore this package has interface Name, which is an interface for the different classes implementing Name. Name exists because javac has a rather sophisticated way of storing and looking up the names of Symbols.

Another one is ElementVisitor, interface for all sorts of scan- and visit classes that can be found in javax.lang.model.util.

#### javax.lang.model.type

_Interfaces used to model Java programming language types._

Here we find, for example, the TypeMirror interface. TypeMirror is the returntype for the asType() method in Element. You use it to get the type of a symbol. TypeMirror is implemented by Type, which is in jdk.compiler.

Another one is TypeVisitor, an interface similar to ElementVisitor. It is the interface for all sorts of scan- and visitor implementations found in javax.lang.model.util.

#### javax.lang.util

_javax.lang.model.util._

Package full of implementation classes of TypeVisitor and ElementVisitor. These scan- and visit classes are use for traversing types and symbols, similarly to implementations used for traversing the Tree objects of the AST.

#### javax.tools

_Provides interfaces for tools which can be invoked from a program, for example, compilers._

This package contains interfaces DocumentationTool and JavaCompiler. These interfaces have their implementations in jdk.compiler. The class ToolProvider has method `getSystemJavaCompiler()`, which I use when I want my app to compile files.

Another relevant one is SimpleJavaFileObject, a class that you can extend and use to represent files (.java, .class, .html or 'other') in a way that javac can work with them. I wrote about them in [this blog post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2026-02-03-getting-the-files-to-compile.md).

## The jdk.compiler module

Here the implementation of the compiler is found. On the [javadoc page](https://docs.oracle.com/en/java/javase/21/docs/api/jdk.compiler/module-summary.html) we find the documentation, limited to the exported packages. 

### module-info

This line is on top of the module javadoc page:

_The com.sun.source.* packages provide the Compiler Tree API: an API for accessing the abstract trees (ASTs) representing Java source code and documentation comments, used by javac, javadoc and related tools._

There is more than the com.sun.source packages in the module but, unlilke many other packages, these ones are exported. Non-exported packages are not documented on the javadoc webpage.The only other package that is exported without restriction is com.sun.tools.javac.

This is what module-info looks like. It is more extensive than the module-info of java.compiler. 

```
module jdk.compiler {
    requires transitive java.compiler;
    requires jdk.internal.opt;
    requires jdk.zipfs;

    exports com.sun.source.doctree;
    exports com.sun.source.tree;
    exports com.sun.source.util;
    exports com.sun.tools.javac;

    exports com.sun.tools.doclint to
        jdk.javadoc;
    exports com.sun.tools.javac.api to
        jdk.javadoc,
        jdk.jshell;
    exports com.sun.tools.javac.resources to
        jdk.jshell;
    exports com.sun.tools.javac.code to
        jdk.javadoc,
        jdk.jshell;
    exports com.sun.tools.javac.comp to
        jdk.javadoc,
        jdk.jshell;
    exports com.sun.tools.javac.file to
        jdk.jdeps,
        jdk.javadoc;
    exports com.sun.tools.javac.jvm to
        jdk.javadoc;
    exports com.sun.tools.javac.main to
        jdk.javadoc,
        jdk.jshell;
    exports com.sun.tools.javac.model to
        jdk.javadoc;
    exports com.sun.tools.javac.parser to
        jdk.jshell;
    exports com.sun.tools.javac.platform to
        jdk.jdeps,
        jdk.javadoc;
    exports com.sun.tools.javac.tree to
        jdk.javadoc,
        jdk.jshell;
    exports com.sun.tools.javac.util to
        jdk.jdeps,
        jdk.javadoc,
        jdk.jshell;
    exports jdk.internal.shellsupport.doc to
        jdk.jshell;

    uses javax.annotation.processing.Processor;
    uses com.sun.source.util.Plugin;
    uses com.sun.tools.doclint.DocLint;
    uses com.sun.tools.javac.platform.PlatformProvider;

    provides java.util.spi.ToolProvider with
        com.sun.tools.javac.main.JavacToolProvider;

    provides com.sun.tools.javac.platform.PlatformProvider with
        com.sun.tools.javac.platform.JDKPlatformProvider;

    provides javax.tools.JavaCompiler with
        com.sun.tools.javac.api.JavacTool;

    provides javax.tools.Tool with
        com.sun.tools.javac.api.JavacTool;
}
```

Note that there is a long list of restricted exports to other jdk modules. Most of them have a name starting with com.sun.tools.javac. The only package with this prefix is package com.sun.tools.javac itself.

### Packages in jdk.compiler/com.sun.source

#### com.sun.source.doctree

_Provides interfaces to represent documentation comments as abstract syntax trees (AST)._

The package has only interfaces, of which the root one is DocTree. Although I have not familiarized myself with 'comments as abstract syntax trees', I notice that DocTree is an interface for all sorts of DCTree objects that live in the com.sun.tools.javac.tree.DCtree class. DCTree is a large class with static inner classes representing all sorts of derivative classes. It is very similar to JCTree.

Btw JCTree also lives in com.sun.tools.javac.tree, as com.sun.tools.javac.tree.JCTree.

#### com.sun.source.tree 

Equivalent of the previous package. many interfaces, one for each type of AST node. Tree is the central interface from which others are derived. 

An interesting one is Scope, which is an interface for com.sun.tools.javac.api.JavaScope. Scopes are used in combination with Symbols and help to resolve them. Scope know what symbol declarations are in them and they know their enclosing scope. If they cannot find the symbol declaration of a symbol being used, javac will traverse up the scope hierarchy until it finds the declaration. 

#### com.sun.source.util

Here we find more classes than interfaces, a lot of them important for my purposes. They are basic for starting the compilation process, for managing compilation tasks and for working with AST's (visiting, access to JCTree implementations). These are the ones I use:

```
JavacTask
TaskEvent
TreeScanner
TreePathScanner
TreePath
Trees
SimpleTreeVisitor (non-recursive, visiting of single Tree objects)
```

Furthermore there are equivalent classes for JavaDoc work. Think of DocTreeScanner, DocTreePathScanner and DocTrees.

### Packages in jdk.compiler/com.sun.tools.javac

This branch has 14 packages in it. It is a package in itself as well, as it contains a class named Main. com.sun.tools has another package in it, namely com.sun.tools.doclint, of which I do not know the function.

#### com.sun.tools.javac.api

These classes are often implementations and extensions of classes found in com.source.util. It contains JavacTool, which is the implementation of JavaCompiler, an interface in java.compiler/javax.tools. You create a JavaCompiler at the start of compilation and you need to generate a JavacTask from it. JavacTool is being provided in the module-info file and is the implementation of JavaCompiler, which is the interface that is declared as 'uses' in the module-info file of java.compiler.

A list with short descriptions:

```
BasicJavacTask              extends com.sun.source.util.JavacTask
ClientCodeWrapper           no extends/implements. Wrap objects to enable unchecked exceptions to be caught and handled.
DiagnosticFormatter         interface, extends javax.tools.Diagnostic<S>
Entity                      class, table of entities defined in HTML 5.2. HashMap with many preset html terms.
Formattable                 interface, about formatting, Locale, toString, for javac classes that have non-trivial formatting needs
JavacScope                  implements com.sun.source.tree.Scope
JavacTaskImpl               extends BasicJavacTask, see first item
JavacTaskPool               class. Pool of reusable JavacTasks. Might make compiling more efficient.
JavacTool                   Implementation of javax.tools.JavaCompiler.
JavacTrees                  Implementation of com.sun.source.util.Trees, which is an abstract class. Via DocTrees, which is abstract extension of Trees.
Messages                    Interface for com.sun.tools.javac.util.JavacMessages.
MultiTaskListener           Implements com.sun.source.util.TaskListener.
WrappingJavaFileManager     Wraps all calls to a given file manager. Extends ForwardingJavaFileManager from javax.tools, which implements JavaFileManager form javax.tools.
```

#### com.sun.tools.javac.code

I Have a hard time figuring out what the classes in this package have in common. I was surprised to find a Scope class, which seems to be the class defining what a scope is or at least what I think it is. This Scope class does not implement any interface, nor does it extend any class. It is an abstract class which is extended by its static inner classes, that have names like ErrorScope, ImportScope and WritableScope.

NOTE: I asked ChatGPT who explained that JavaScope is a bridge between Scope interface (com.sun.source.tree.Scope) and com.sun.tools.javac.code.Scope. The three types are connected, they work on the same problem. I found references in JavacScope to specofic static inner classes in com.sun.tools.javac.code.Scope.

ChatGPT remarked that there is not a uniform implementation of Scope, just as there isn't for Tree. This explains the static inner classes in the implemented abstract Scope class.

Apart from Scope, this package has two important classes, namely Symbol and Type. Both are abstract with implementations of themselves in inner static subclasses. 

Furthermore there is SymTab class, _defining all predefined constants and operators as well as special classes such as java.lang.Object, which need to be known to the compiler._ There is Types, a static utility class for working with Type objects. There are also classes related to annotations (like SymbolMetaData) and classes like Flags and Kinds, defining all sorts of constants and enums.

Generally, this package is about working with individual Scopes, Types, Symbols and Annotations.

#### com.sun.tools.javac.comp

Every class, or most classes, seem to represent a compilation pass. Names are strongly suggestive of this, sparse javadoc comments as well. Think of class names like Attr, Enter, LambdaToMethod and MemberEnter.

#### com.sun.tools.javac.file

Here we find implementation classes related to the file management. Think of BaseFileManager and JavacFileManager. 

There are 8 classes in total, other names ar eLocations, PathFileObject and RelativePath.

#### com.sun.tools.javac.jvm

Classes having to do with bytecode, classfiles and the JVM. At a certain point the .class files must be generated and I think the tooling lives here.

#### com.sun.tools.javac.launcher

Contains a single class named Main. Javadoc says: _Compiles a source file, and executes the main method it contains._

#### com.sun.tools.javac.main

Plays a role when compiling via the command line, at least I think so. Has an Arguments class and a Main class. Seven files, one being an enum.

#### com.sun.tools.javac.model

I am triggered by this one, it has four classes, three of them utility classes that operate on Types and Elements. The names of the classes with their descriptions are:

```
AnnotationProxyMaker     A generator of dynamic proxy implementations of java.lang.annotation.Annotation.
FilteredMemberList       Utility to construct a view of a symbol's members, filtering out unwanted elements such as synthetic ones.
JavacElements            Utility methods for operating on program elements. Implements javax.lang.model.util.Elements.
JavacTypes               Utility methods for operating on types. Implements javax.lang.model.util.Types.
```

#### com.sun.tools.javac.parser

Claases about parser- and token things. Names like JavacParser, JavacTokenizer, Scanner and Tokens.

#### com.sun.tools.javac.platform

Dealing with versions of the JDK platform I think. Only four files in it, two of them being interfaces implemented by one of the two classes, JDKPlatformProvider.

#### com.sun.tools.javac.processing

Implementations of interfaces found in javax.annotation.processing. This has to do with annotation processing. Only seven files inside.

#### com.sun.tools.javac.resources

All these classes have long lists with all sorts of messages in them, stores in multidimensional arrays, for different languages.

#### com.sun.tools.javac.tree

Here the JCTree class is found, very important for my purposes. DCTree, representing the parallel structure for Javadoc, is also present.

What surprised me was the TreeScanner class found here. TreeScanner exists in com.sun.source.util as well and that is actually the class that I use (or its extension TreePathScanner). ChatGPT explained to me that this TreeScanner is an internal one that operates not on Tree objects but on the specific implementations, which makes it more efficient.

On my question if it would benefit me to use this 'internal' TreeScanner ChatGPT answered that it wouldn't, it is mainly used if you write 'deeper' applications. 

Note that when I use the public TreeScanner (or TreePathScanner) I can still get to the internals of the compiler. When I want to have a Symbol type instead of the more superficial Tree type, I just cast it. The line below is from a overridden VisitClass I created:

```
Symbol.ClassSymbol currentClass =  (ClassSymbol) trees.getElement(getCurrentPath());
```

This works well for me so no need to use the internal TreeScanner. Btw this internal package also includes a TreeMaker class, which is expected given that this is what a compiler must do.

One more thing: the package contains TreeInfo, a utility class with a ton of 'is' methods that help to understand the character of a specific Tree. I might want to use that in the future.

#### com.sun.tools.javac.util

Many utility classes plus some Exception classes. They do some fundamental operations. I recognized the Name class (very often used, important because javac deals with Symbol names in an advanced way), the Names class (access to the name table), a Position class (dealing with LineMap, line and column etc).

I also see a List class, a specific implementation of AbstractCollection and java.util.List. This class represents a specific sort of linked list. As the javadoc says: _List is the main container class in GJC._ There is also a class named Pair, _a generic class for pairs._

Summary: here all sorts of basic building blocks live, and most often they are not implementations of classes or interfaces living elsewhere in java.compiler or jdk.compiler. This means that they are mainly for internal use.

### jdk.internal.shellsupport

#### jdk.internal.shellsupport.doc

Only two classes. Described as _A javadoc to plain text formatter_ and _Helper to find javadoc and resolve @inheritDoc._

#### jdk.internal.shellsupport.doc.resources

Only one tiny final class named javadocFormatter (note the lower case, I don't know why). It is an extension of java.util.ListResourceBundle.

### sun.tools.serialver

#### sun.tools.serialver

Two small classes: _Supporting class for the serialver tool_ and _Utility for integrating with serialver and for localization. Handle Resources. Access to error and warning counts. Message formatting._ They live in the same file. Don't know what they are used for.

#### sun.tools.serialver.resources

Four classes containing data/warnings in different languages, unmimportant. 

## Conclusion

There is a lot of stuff in the Java compiler but it is finite. I'm getting a better view of it, although there are plenty of areas that I am not familiar with yet. Annotations and JavaDocs, both seem to play a substantial role in javac, are two of these.

In the next blog I will try to summarize the info I have collected. I need a simplified but accurate mental model of the contents of the two modules so that I will have the freedom to work with all the available tools in an effective and creative way.



