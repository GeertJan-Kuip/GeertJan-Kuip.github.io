# Elements of coding style

It has been a while since I wrote my last blog post and the reason for my absence is me being immersed in my project. Mainly writing and structuring code, it's progressing fine.

While coding, I found myself applying patterns, many of which I learned from the java.compiler and jdk.compiler library, from the Clean Architecture book (see a previous post about it) or from I don't know where. In this post I'll describe some of these patterns, including my motivation for using them.

## Child classes as nested inner classes

The java implementation classes `com.sun.tools.javac.tree.JCTree` and `com.sun.tools.javac.code.Symbol` (among others) are abstract and have their subclasses as non-abstract static inner classes inside them. It makes these source files rather large but it has the benefit of providing all code in one place, immediately seeing the relationships between abstract parent and instantiatiable children.

I applied this pattern myself in my custom `Node` class. Node objects represent everything that can be drawn as a node/vertex in a uml diagram, ie a class, a package or a module. As different types of nodes will share some identical properties (like 'label', 'description' or 'id'), they each have their custom fields as well. Therefore I applied this structure:

```
public abstract class Node {

  //fields common to all Node subtypes

  private Node(){}

  private Node(String id, Point2D pos1, Point2D pos2, String label, String description) {
    this.id = id;
    this.pos1 = pos1;
    this.pos2 = pos2;
    this.label = label;
    this.description = description;
  }

  public static class ClassNode extends Node{

    //custom fields

    public ClassNode(){
      super();
    }

    public ClassNode(
                String id, Point2D pos1, Point2D pos2, String label, String description,
                ClassNodeKind type, int directDeps,
                 int directPlusTransDeps, int rank, int invDirectDeps, boolean hasExternalDeps,
                 boolean isDepForExtClasses, int nExtDeps, int nExtInverseDeps,
                 boolean isRepInst, int nNonStaticFields, int nStaticMethods,
                 List<CreationTranslated> creationSites, List<CreationTranslated> createsSites){

        super(id, pos1, pos2, label, description);
        this.type = type;
        this.directDeps = directDeps;
        this.directPlusTransDeps = directPlusTransDeps;
        this.rank = rank;
        this.invDirectDeps = invDirectDeps;
        this.hasExternalDeps = hasExternalDeps;
        this.isDepForExtClasses = isDepForExtClasses;
        this.nExtDeps = nExtDeps;
        this.nExtInverseDeps = nExtInverseDeps;
        this.isRepInst = isRepInst;
        this.nNonStaticFields = nNonStaticFields;
        this.nStaticMethods = nStaticMethods;
        this.creationSites = creationSites;
        this.createsSites = createsSites;
      }
    }
  }

  public static class PackageNode extends Node{

    // analog to ClassNode

  }

  public static class ModuleNode extends Node{

    // analog to ClassNode and PackageNode

  }
}
```

The benefit of this is that by using `super` in the constructor of the subclasses I prevent duplication of code. Things beocme easier to maintain. And of course I can now use abstract type `Node` on all occassions where it does not matter whether the Node is a ClassNode, a PackageNode or a ModuleNode.

## Generics

Speaking about using `Node` as abstract representation for different implementation types, one of the classes where this comes in handy is the class that is responsible for doing graph calculation on a set of Node objects and a set of Edge objects. Library [JGraphT](https://jgrapht.org/) can use any type of object as node (btw it's own terminology has the term 'vertex' instead of 'node').  

I made some of the types that I use as input for JGraphT generic, in a rather specific way. This is the list of fields at the start of my ` JGraphComputations` class:

```
public class JGraphTComputations {

    SymbolRegistry registry;
    Set<? extends Symbol> relevantClasses;
    Set<Pair> relevantPairs;
    List<Edge> edges;
    List<? extends Node> nodes;

    Graph<Symbol, DefaultEdge> graph;
    Graph<Symbol, DefaultEdge> invertedGraph;
```

Note how similarly I treated the set of Symbol objects and the list of Node objects, both using generics. 

The `Set<Pair> relevantPairs` has the Pair class, which I also gave a distinct generic style:

```
public record Pair<S1 extends Symbol, S2 extends Symbol>(S1 from, S2 to) {
}
```

It is very convenient that I can work with any kind of Node or Symbol now without having to cast them to their abstract type or vice versa.

## Failing fast

Using `Objects.requireNonNull(Obejct o)` at the start of a method to check for null input is really helpfull, as it immediately shows you where an illegal null argument was found in the error stack trace. I used it in this method constructor for example:

```
    public JGraphTComputations(SymbolRegistry registry, Set<? extends Symbol> relevantSymbols, Set<Pair> relevantPairs, List<Edge> edges, List<? extends Node> nodes) {
        Objects.requireNonNull(registry);
        Objects.requireNonNull(relevantSymbols);
        Objects.requireNonNull(relevantPairs);
        Objects.requireNonNull(edges);
        Objects.requireNonNull(nodes);

        this.registry = registry;
        this.relevantClasses = relevantSymbols;
        this.relevantPairs = relevantPairs;
        this.edges = edges;
        this.nodes = nodes;
        createGraph();
        createInvertedGraph();
    }
```

Throwing RuntimeException objects on non-valid input is also great, in general it is preferable to validate input as soon as it enters your process and to ring all alarm bells when there is something wrong with it. For example I did this:

```
    public UmlModuleDataObject getUmlModuleDataObject(String symbolIdentifier){

        if (registry.getStringSymbol().get(symbolIdentifier)==null){

            throw new RuntimeException(ColorOutput.YELLOW + "The requested symbol identifier " + symbolIdentifier
                    + " cannot be found in the symbol registry" + ColorOutput.RESET);
        }
```

Any scenario in which such an illegal null value is propagated is to be avoided, and writing the custom message (in yellow) immediately tells me what went wrong.

## Dependency inversion

I already wrote about this topic [earlier](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2026-05-26-clean-architecture.md#dependency-inversion-principle), and I think right now I have a sort of deeper understanding of how and why this is such a good concept. 

In my appllication I decided that my package `symbolanalysis`, that is responsible for the compilation of the source files and the generation of sets of `Relation` objects that describe relationships between classes as pairs of Symbol objects, should be independent. But as this `symbolanalysis`, that lives at the core of the application, requires files to compile, it is naturally dependent on the package that is respnible for the management of the source files. 

What I did was creating an interface 'JavaFilesSupplier' in the independent `symbolanalysis` package. It has only one method, which makes it very convenient to use:

```
package com.geertjankuip.astvisitor.symbolanalysis;

import java.util.List;

public interface JavaFilesSupplier {

    public List<JavaFile> supply();
}
```

This interface is implemented by class `JavaFilesContainer` that lives in the `filemanagement` package. Thus interface and implementation are in different packages. This is the first part of its code:

```
package com.geertjankuip.astvisitor.filemanagement;

import com.geertjankuip.astvisitor.symbolanalysis.JavaFile;
import com.geertjankuip.astvisitor.symbolanalysis.JavaFilesSupplier;

import java.util.List;

public class JavaFilesContainer implements JavaFilesSupplier {
```

One thing important: if you want to make a package or module independent and use dependency inversion, you must make sure that every type you use in the interface (either method arguments or return values) is part of the independent package/module as well. In this case, `JavaFile` must live in the independent module. If it would live in `filemanagement`, packages `filemanagement` and `symbolanalysis` would be each other's dependencies. It is explained clearly in Clean Architecture, that arrows between modules must all point in the same direction.

## Using existing Java classes instead of creating from scratch

The interface JavaFilesSupplier has class JavaFile as its dependency. Initially I created my own JavaFile type, just being a simple class containing file name and a String containing the code of the file. 

This worked until I wanted to use a list of such file objects as input for the javac compilation process. Here a specific object type is required, namely `<? extends JavaFileObject>`. JavaFileObject is a very generic interface representing a file, a child of interface `FileObject`. 

My first approach was to create a child of JavaFileObject and make my own JavaFile  class a field in it, but that made for a somewhat convoluted solution. The proper way was simply to extend JavaFileObject (or actually its native Java subclass SimpleJavaFileObject) and add a field to it containing the String value of the source file. By aligning my goals with existing Java types complexity got reduced, and my understanding of Java increased. If people need to understand my code in the future and if they are already familiar with the JavaFileObject interface, they will not be forced to learn about my incidental custom objects.

That's it for now. 





