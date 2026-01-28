# Visitor and Abstract Syntax Tree

The design pattern I find most difficult to work with is the Visitor pattern, the last pattern described in "Design Patterns" of the Gang of Four. Understanding Visitor is sort of required to understand how you can work with the abstract syntax trees that are creating when Java parses source code. As I want to be able to create tooling that helps to easily find structure in source code, I need to be able to work fluently with AST's. 

## What the Visitor pattern is good for

_Design Patterns_ describes the following objective of the Visitor pattern:

_"Represents an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates."_

The book immediately uses the example of AST's to clarify the concept, meaning that the two are sort of naturally connected. In Java, Abstract syntax trees are collections of Java  objects that implement the Tree interface, directly or indirectly. There are [many subinterfaces](https://docs.oracle.com/javase/8/docs/jdk/api/javac/tree/com/sun/source/tree/Tree.html) but in the end, every element in the AST can be of type Tree. 

## Single dispatch, @Override and method overloading

Java needs the Visitor pattern because it is a single dispatch language. The word 'dispatch' is about the question how and when Java decides which method should be applied to an object whose runtime type differs from its reference type. For example, when I have the following declaration:

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

- An object structure consisting of elements that have different runtime types but can all be accessed by the same reference type (interface, superclass)
- Some sort of method(s) that can traverse all these methods
- A separate Visitor class that contains 




```
List<String> myList = new ArrayList<>();
```