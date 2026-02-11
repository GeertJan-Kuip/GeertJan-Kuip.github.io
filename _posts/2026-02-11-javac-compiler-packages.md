# Javac compiler packages

Working with compiler internals means working with two modules, namely platform module java.compiler and JDK implementation module jdk.compiler. The former, according to the javadoc of its module-info file, _defines the Language Model, Annotation Processing, and Java Compiler API's_. The latter _defines the implementation of the system Java compiler and its command line equivalent, javac._

In this blog post I'll discuss the contents of both modules, not exhaustive but enough to get some idea what jpackages are in it and how these packages can be utilized.

## The java.compiler module

The java.compiler module is completely open, meaning that every package is exported. Using them as imports in your code will not force you to compile and/or run the 

### The module-info.java file, declaring services

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

### What is in java.compiler






