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

This is how an override looks like:

```
public class MyCustomScanner extends TreeScanner<Void,Void> {
    
    // insert overrides here
}
```

### Overriding a visit method

TreeScanner contains many visit methods and you can override one or more of them to insert custom code. This is a very basic example:

```
    @Override
    public Void visitClass(ClassTree node, Void p) {
        System.out.println("Class: " + node.getSimpleName());
        return super.visitClass(node, p);
    }
```

The interesting thing is the return statement. In the previous blog post we saw that traversal of the tree required that the visit methods contained 'get' methods to provide new child trees to scan. If you simply override a visit method without keeping this get methods intact, the traversal will stop which is probably not what you want. The return statement calls the implementation of the parent method in TreeScanner, and as that parent method has all the get methods, traversal will proceed as normal. Basically the only thing that is added to the traversal process is the line `System.out.println("Class: " + node.getSimpleName());`.

Other remarkable elements in this overridden method are the return type (`Void`) and the second argument, `Void p`. We will get back to that later.

### Overriding the scan method

Below is an overridden scan method. Like the overridden visit method, it has a return statement with 'super.scan' in it, which is just meant to not interrupt the traversal process. The `getKind` method is one of two methods found in interface Tree, the other being `accept`. These are the methods applicable to any Tree, while child interfaces have extra methods fitting their specific role and characteristics.

```
    @Override
    public Void scan(Tree tree, Void p) {
        if (tree != null) {
            System.out.println(tree.getKind() + ":" + tree.getClass().toString());
        }
        return super.scan(tree, p);
    }
```

### Overriding the reduce method

The reduce method can as well be overridden. The default is this, it does not reduce anything as r2 is not used:

```
    public R reduce(R r1, R r2) {
        return r1;
    }
```

To understand what the role is of the reduce method, let's look at the scanAndReduce method for single Tree objects:

```
    private R scanAndReduce(Iterable<? extends Tree> nodes, P p, R r) {
        return reduce(scan(nodes, p), r);
    }
```

The second value is the aggregated value of previous scans, while the first is the value returned by the current scan.

If you override, you must account for the fact that any of the two arguments can be null. ChatGPT advised me to start any override of reduce like this:

```
    if (r1 == null) return r2;
    if (r2 == null) return r1;
```

This implicates that the return value of reduce is only null if both r1 and r2 are null. It also means that no matter how many of the scan and scanAndReduce methods return null, it is still possible to get a non-null aggragate result.

To get a non-null result, it is required to tell Java what the return type must be, ie provide a specific type for the generic type parameter R. Furthermore it is required to provide code in the overridden reduce method that combines r1 and r2. Lastly, at least one of the visit methods must return a non-null value and have a signature with a type equivalent to the return type of the reduce method.

