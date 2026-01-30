# Visitor and Abstract Syntax Tree

The design pattern I find most difficult to work with is the Visitor pattern, the last pattern described in "Design Patterns" of the Gang of Four. Understanding Visitor is sort of required to understand how you can work with the abstract syntax trees that are creating when Java parses source code. As I want to be able to create tooling that helps to easily find structure in source code, I need to be able to work fluently with AST's. 

## What the Visitor pattern is good for

_Design Patterns_ describes the following objective of the Visitor pattern:

_"Represents an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates."_

The book uses the example of AST's to clarify the concept, meaning that the two are sort of naturally connected. In Java, Abstract syntax trees are collections of Java  objects that implement the Tree interface, directly or indirectly. There are [many subinterfaces](https://docs.oracle.com/javase/8/docs/jdk/api/javac/tree/com/sun/source/tree/Tree.html) but in the end, every element in the AST can be of type Tree. 

## Single dispatch, @Override and method overloading

Java needs the Visitor pattern because it is a single dispatch language. The word 'dispatch' is about how and when Java decides which method should be applied to an object whose runtime type differs from its reference type. For example, when I have the following declaration:

```
Animal cat = new Cat();
```

Assuming that the Cat class inherits form the Animal class, the reference type is `Animal`, while the implemented runtime type is `Cat`. Now in the case that a method is called on `cat`, and there exist two variants of this method, one in Animal and one override in Cat, Java needs to decide which one to use. It will use the method of Cat, meaning that the choice of method is only resolved during runtime, not compile time. During compile time Java can only see that cat is of type Animal, which is its reference type.

Now lets say that we have two cats with different reference types but the same runtime type:

```
Animal cat1 = new Cat();
Cat cat2 = new Cat();
```

And let's say that there is some class, say `AnimalClinic`, that has multiple overloads for a method named `sterilize`:

```
class AnimalClinic{

  public void sterilize(Animal animal){...}
  public void sterilize(Cat cat){...}
  public void sterilize(Dog dog){...}
}
```

Now if we would apply the sterilize method on both cats, each would be sterilized but they would not be sterilized by the same method. For cat1, the first method `sterilize(Animal animal)` will be used, for cat2 `sterilize(Cat cat)` will be used. The reason is that, unlike in the case of overridden methods, Java will resolve the choice of method based on the reference type during compilation, not on the runtime type during runtime. 

There is thus a difference in the way Java resolves overridden methods and overloaded methods. The first is resolved during runtime based on the runtime type, the second is resolved based on the reference type during compile time. Java is a single dispatch language because it only resolves overridden methods during runtime. It would be a double dispatch language if it would resolve overloaded methods as well during runtime.

The Visitor pattern is sort of complex and would not be necessary if Java would be a double dispatch language. But it isn't. 

## How Visitor works

For Visitor to work in Java the following ingredients must be available:

- An object structure consisting of elements that have different runtime types but can all be accessed by the same reference type (interface, superclass).
- An `accept(Visitor visitor)` method that is inherited and implemented by the elements of the object structure.
- Some process that traverses the object structure, calling the `accept` method on any of the elements.
- A separate Visitor class that contains overloaded `.visit(Element e)` methods for all the different runtime types in the object structure.

### A basic implementation

A basic implementation of the Visitor pattern can look like this. To clarify our intentions, we create two interfaces, one for the type of element that forms the basis of the aggregate object, and one for the visitor:

```
public interface CarPart{
    public void accept(Visitor visitor);
}

interface Visitor{
    public visitEngine(CarEngine carEngine);
    public visitDoor(CarDoor carDoor);
}
```

Next we provide the implementations for the CarPart elements. For brevity I limit myself to two different implementations:

```
public class CarEngine implements CarPart{

    int horsePower;
    double weight;
    
    public void accept(Visitor visitor){
         visitor.visitEngine(this);
    }
}

public class CarDoor implements CarPart{

    String color;
    DoorKind doorKind;
    
    public void accept(Visitor visitor){
         visitor.visitDoor(this);
    }
}
```

In these classes I have omitted the constructor and the getters and setters but by adding some instance field I hope to make clear that the different CarPart implementations are really different and need to be treated as such.

The next thing is the Visitor class, which implements the Visitor interface. 

```
public class FirstCarVisitor implements Visitor{

    @Override
    public visitEngine(CarEngine engine){

        System.out.println("Power: " + engine.horsepower);
        System.out.println("Weight: " + engine.weight);
    }

    @Override
    public visitDoor(CarDoor door){

        System.out.println("Color: " + door.color);
        System.out.println("Kind of door: " + door.doorKind);
    }
}
```

The essential thing is that there is a dedicated method for every specific CarPart object. In this example I have given each of them its own name, but it is also possible to name them the same so that the only difference between them is the argument type. The methods are then overloads of each other. 

The last class to provide is the Car class. This class is the container that contains and structures all the different CarPart objects. It is good practice to have the Car class contain the method(s) needed to traverse the object structure. In this example the structure is very simple (just a list) but more tree-like structures are possible, and often more appropriate.

```
public class Car{

    List<CarPart> carParts = new ArrayList<>();

    public Car(CarPart engine, CarPart door){
      
        carParts.add(engine);
        carParts.add(door);
    } 

    public void traverse(Visitor visitor)}
        
        for (CarPart carPart : carParts){
            carPart.accept(visitor);
        }
    }
}
```

And now the code to run the Visitor (assuming a Car object with all the parts has been created):

```
car.traverse(new FirstCarVisitor());

# sample output:
Power: 97
Weight: 145.8
Color: green
Kind of door: left front door
```

Upon running this, every CarPart in the Car object will have its acccept method being called with FirstCarVisitor as argument. Because FirstCarVisitor has a dedicated visit method for every CarPart, every CarPart will be affected in an appropriate way. To have other visit methods applied, just create another implementation of the Visitor interface and create a new set of visit methods.

### Variations

You can do things slightly differently, for example by not giving every visit method in FirstCarVisitor a different name. You can call them all just visit, the overloading will make sure that every CarPart will have the right overload selected, because the argument of every visit method tells which CarPart it should be applied on. That is the magic that happens because of 'this' in `visitor.visit(this).`

What you can also do is provide empty default implementations for every visit method in the Visitor interface. When you want to create a Visitor implementation but are only interested in applying visit methods on a subset of the CarPart elements (only the doors but not the engine for example), you only have to write a visitDoor method and you won't have to bother about the visitEngine method.

## Visitor and Abstract Syntax Tree

As mentioned, an important target for the Visitor pattern in Java is the Abstract Syntax Tree. The visitor implementation  used for this is more complex than the example discussed before. 

What makes the AST special is that it is not stored in a single object like Car. The AST in Java is a collection of Tree objects that contain references to parent nodes and child nodes. Traversal of the tree structure is not stored in some method, but is done with a recursive process. The code jumps from the root branch (CompilationUnitTree is the highest node in the hierarchy but you can start at a lower level), calls its accept method, then calls the methods that provide the references to the different types of children, calls their accept methods, calls the methods that provide the references to their children, calls their accept methods etcetera. 

This means that the traversal of the AST happens in a structured way. The nodes of the Tree are of a great number of subtypes and subsubtypes, and every type has its own defined set of children. 

### Tree objects and their JC-implemenations

The implementation of this is rather extensive. There is a hierarchy of Tree interfaces, which are conveniently accessible, and behind these interfaces you can find classes that contain the under-the-hood implementations. These classes live in the 3500-line JCTree class (com.sun.tools.javac.tree) as static inner classes and have names like JCCompilationUnit, JCVariableDecl, JCClassDecl, JCBlock etc. The JCTree class is not meant for normal use, this is the disclaimer in the Javadoc comment:

_This is NOT part of any supported API. If you write code that depends on this, you do so at your own risk. This code and its internal interfaces are subject to change or deletion without notice._

To be able to access is you need an --add-export flag when you run or compile the code. To understand the specifics of Tree traversal you can get ahead by reading the code of the Tree interfaces (like ClassTree, BlockTree etc). You will see get-methods that provide access to the Tree elements that are considered valid children, given the way Java AST's are being structured.

### Traversal of the AST

As the Java Abstract Syntax Tree cannot be found in a single file but only exist as a collection of Tree objects referencing each other (implemented in obscure JCTree objects), the traversal of the AST is done in a rather sophisticated way that I found hard to figure out. Traversal lives in the Visitor class, which in Java is called TreeScanner. TreeScanner implements TreeVisitor, an interface that contains all abstract Visit methods with signatures like `R visitUnary(UnaryTree node, P p);`. I counted more than 60 methods, each for a specific Tree subtype.

The implementations of the TreeVisitor methods live in TreeScanner, and look like this:

```
    @Override
    public R visitCompilationUnit(CompilationUnitTree node, P p) {
        R r = scan(node.getPackage(), p);
        r = scanAndReduce(node.getImports(), p, r);
        r = scanAndReduce(node.getTypeDecls(), p, r);
        r = scanAndReduce(node.getModule(), p, r);
        return r;
    }

    @Override
    public R visitEnhancedForLoop(EnhancedForLoopTree node, P p) {
        R r = scan(node.getVariable(), p);
        r = scanAndReduce(node.getExpression(), p, r);
        r = scanAndReduce(node.getStatement(), p, r);
        return r;
    }
```

As you see, every method body contains the methods 'scan' and 'scanAndReduce'. The scan method has two overloads, one for a single Tree and one for an iterable of Trees. In both cases, the accept method is called on the Tree:

```
    public R scan(Tree tree, P p) {
        return (tree == null) ? null : tree.accept(this, p);
    }

    public R scan(Iterable<? extends Tree> nodes, P p) {
        R r = null;
        if (nodes != null) {
            boolean first = true;
            for (Tree node : nodes) {
                r = (first ? scan(node, p) : scanAndReduce(node, p, r));
                first = false;
            }
        }
        return r;
    }
```

While the scan method is used for the first child of a Tree node, the scanAndReduce method is used for subsequent Tree nodes. Reduction can be used to aggragate the return values of the scan method, and that return value is the value that is returned when calling the accept method on a Tree. These are the overloads of scanAndReduce:

```
    private R scanAndReduce(Tree node, P p, R r) {
        return reduce(scan(node, p), r);
    }

    private R scanAndReduce(Iterable<? extends Tree> nodes, P p, R r) {
        return reduce(scan(nodes, p), r);
    }
```

As you see both methods are private. Contrary to the scan method, you will not call these methods from the outside. The first scanAndReduce is used for a single Tree node, the second for an iterable of Tree nodes. Both implementations are a combi of the scan and the reduce method. The reduce method, the last method that doesn't have visit in the name, is this:

```
    public R reduce(R r1, R r2) {
        return r1;
    }
```

If we go back to the implementation of the visit methods, we see that they have their own Tree type as argument, plus a generic type P as second argument. This second 



    @Override
    public R visitNewClass(NewClassTree node, P p) {
        R r = scan(node.getEnclosingExpression(), p);
        r = scanAndReduce(node.getIdentifier(), p, r);
        r = scanAndReduce(node.getTypeArguments(), p, r);
        r = scanAndReduce(node.getArguments(), p, r);
        r = scanAndReduce(node.getClassBody(), p, r);
        return r;
    }



What the accept method returns depends on the Visitor implementation you provide. If you do not override 


get methods that 



