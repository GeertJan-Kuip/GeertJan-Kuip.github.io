## super(), abstract classes and constructors

Classes extending other classes do not inherit constructors by default but they can reuse the body of a parent constructor by using **_super()_** in their own constructor body. While trying to understand the super() method better, I learned something about constructor inheritance.

### What I learned about constructor inheritance

If a subclass wants to define a new constructor with a signature that is different from the signatures in its parent's body, it can only do so if the parent has a 'no argument' constructor among its constructors. A few things are important here:

- the default constructor of a class is the zero argument constructor, that sets all instance fields to their default values (0, null).
- adding a constructor to a class body means the default constructor will be overridden.
- if a subclass wants to add its own constructor, it can only do if the parent class either has zero constructors, in which case the parent indeed has a (default) zero argument constructor, or if the parent has one constructor, in which case this must be a zero argument constructor, or if the parent has multiple constructors, one of them must be a zero argument constructor.
- Thus: the introduction of a new constructor fails if the parent class lacks a zero argument constructor.

### Using super()

Constructors are not inherited but you can use the constructors of the parent class by invoking **_super()_** in the constructor body. That constructors are not inherited is logical in the light of naming: the constructor is named after the class so is not identical to the constructor of the parent class.

The **_super()_** method can also be used to partly inherit from the parent class. If the parent class has a constructor with one argument and you want to create a constructor with an second argument, you can call _super(firstargument)_ in the constructor body to import the body of the parental one-argument constructor. You can add extra code to deal with the second argument.

### Code example 

Below some code with three levels of classes in a hierarchical tree. This compiles and I'll get back to the interface later:

```
interface Calculating {
    Number addNumbers(Integer i, Integer j);
}

abstract class SuperSuperClass implements Calculating {
    int a,b;

    public SuperSuperClass(){}	// zero argument constructor: required so subclasses can add extra constructors
    public SuperSuperClass(int a){ this.a=a;}
}

abstract class SuperClass extends SuperSuperClass{
    int c;
    static int statInt = 5;

    public SuperClass(){} // again zero argument constructor for the same reason
    public SuperClass(int a){ super(a);} // using super() imports parent's version
    public SuperClass(int a, int b, int c){ super(a); this.b=b; this.c=c;} // using super() partially
}

public class NormalClass extends SuperClass {

    // no zero argument constructor required 
    // because there are no subclasses
    public NormalClass(int a){ super(a);} 
    public NormalClass(int a, int b){ this.a=a; this.b=b;}
    public NormalClass(int a, int b, int c){ super(a); this.b=b; this.c = c*statInt;} // inherited statInt 

    @Override
    public Integer addNumbers(Integer i, Integer j) { return i+j;}
}
```

This compiles but it wouldn't if SuperSuperClass and SuperClass wouldn't have had zero-argument constructors. 

### Implementing interfaces in abstract classes

When writing the code above I was surprised that I didn't have to implement the addNumbers() method in the body of SuperSuperClass. The rule is that abstract classes do not have to do that, they pass the abstract method of the implemented interface on to their children. The first descendant that is not an abstract class is forced to do the implementation, which is why the implementation is found in NormalClass. This obligation has trickled down through the inheritance tree.

Note that I applied the 'covariant return type' principle by using Integer as a return type in the implementation while using Number in the abstract method declaration of the interface. Note also that an implemented interface method needs the public access modifier, as interface abstract methods are public by default (and you cannot use a more restrictive access modifier when overriding a method). 

### Endnote

I started experimenting with this after realizing that I had never used super() myself and that there might be more about class inheritance that I didn't know. Few points:

- private fields and methods are not accessible in subclass (perplexity told me they are inherited though)
- final methods cannot be overridden in subclasses
- abstract classes and abstract methods both need the modifier _abstract_
- static methods cannot be overridden
- a method cannot be static and abstract
- the first non-abstract subclass must implement all abstract methods of the parent
- an abstract class can extend a non-abstract class
- abstract classes cannot be instantiated
- a class or method cannot be final and abstract
- abstract methods can only be defined in abstract classes and interfaces
- implementing an abstract method in a subclass follows the same rules for overriding a method, including covariant return type, exception declaration etc.