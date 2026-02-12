# Javac compiler packages

Working with compiler internals means working with two modules, namely platform module java.compiler and JDK implementation module jdk.compiler. The former, according to the javadoc of its module-info file, _defines the Language Model, Annotation Processing, and Java Compiler API's_. The latter _defines the implementation of the system Java compiler and its command line equivalent, javac._

In this blog post I'll discuss the contents of both modules, not exhaustive but enough to get some idea what jpackages are in it and how these packages can be utilized.

## The java.compiler module

The java.compiler module is completely open, meaning that every package is exported. Using them as imports in your code will not force you to compile and/or run the 

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

The 'uses' lines mean that the module relies on two services, these services live in another module and are implementations of its own DocumentationTool and JavaCompiler interfaces. The implementation tool lives in the jdk.javadoc module, whose module-info file has corresponding declarations:

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

### Contents of java.compiler

The java.compiler module has only six packages, they are all listed with an exports clause in module-info. They can be recognized by the 'javax' element with which the package name starts. Easy rule of thumb: packages starting with 'javax' belong in java.compiler, all other packages belong to jdk.compiler.

Here follow short descriptions of these packages. The italic text is form the official javadoc:

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




