# Extending TreeScanner

If you want to analyze Java source code, you can get access to them by parsing the .java files and then extending TreeScanner to walk the parse tree. TreeScanner is a class in com.sun.source.util and as this is an exported package, you can work with it without problems. Extending TreeScanner allows you to provide override methods for the scan method, the reduce method and all the visit methods. 

## Traversal of the Abstract Syntax Tree

In the previous blog post we saw how the parse tree is being traversed. The flow oscillates between TreeScanner and the static subclasses of JCTree, summarized as this, whereby scan and visit are found in TreeScanner and accept and get in the JCTree subclass:

```
scan(Tree) -> accept(Visitor) -> visit(Tree) -> node.get() -> ...
```

In Java's implementation, the scan method has two overloads, one being used if the first argument is of type `Tree` and one being used if the first argument is Iterable<? extends Tree>. This is required because node.get... can return a single Tree or an iterable of Trees.

Furthermore there is a scanAndReduce method, which exist in parallel variants and is being used because the Java implementation allows for the generation of a return value of scan that aggragates the return values of all trees being visited. This scanAndReduce value is a wrapper around the scan method and has the reduce method in it as well. I will get back to this.

## Parsing files

If you want to extract information from a parse tree, you first need to parse the files you are interested in. This is code for it:

```
  this.compiler = ToolProvider.getSystemJavaCompiler(); // gets the implemented compiler via a sort of service principle

  this.listJavaFileObjects = javaFiles.stream().map(x->x.getJavaFileObject()).toList(); // code specific for my project

  // create a task. The last argument is the list of files to parse
  this.task = (JavacTask) compiler.getTask(
        null,
        null,
        null,
        List.of("-proc:none"), // disable annotation processing
        null,
        listJavaFileObjects
  );

  // this.cuTrees is an iterable of CompilationUnitTree objects
  // task.parse creates AST's, analyze does subsequent steps, generating Symbol and Type objects
  try {
      this.cuTrees = task.parse();
      task.analyze();
  } catch (IOException e) {
      throw new RuntimeException(e);
  }

  // This trees object is necessary if you want to work with Symbols and Types
  this.trees = Trees.instance(task);
```

If you only want to work with the parsed tree, and are not interested in Symbol and Type objects, the result that you need from the code above is only the iterable of CompilationUnitTree (Iterable<? extends CompilationUnitTree>). You can omit `task.analyze` and `this.trees = Trees.instance(task);`.

## Extending TreeScanner

TreeScanner is the class that orchestrates the traversal of the AST's. If you create an instance of it and call `scan` on it with the CompilationUnitTree object of your file as first argument, all Tree nodes will be visited. As the visit methods do not do anything with the Trees, it does not give you any result. If you do want a result, ie  extraact information from the AST, you need to create a subclass of TreeScanner and overwrite its methods. The candidate methods to overwrite are scan, visit and reduce.

## Overriding visit




