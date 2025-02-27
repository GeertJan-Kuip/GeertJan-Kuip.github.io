## Abstract classes vs interfaces


Both abstract classes and interfaces can be used as a type variable, enabling polymorphism, and both can force classes that inherit from them to implement certain methods. What makes this topic a bit more difficult is that both can do more than just providing abstract methods. As the number of things you can do in a class is quite substantial, I'll try to provide a full overview of everything that is possible with a regular class and compare that with the possibilities and limitations of abstract classes and interfaces.

A class can be parameterized with the following:

- the access modifier of the class itself (public, private, protected or package-protected)
- an added 'final' modifier for the class
- the 'extend' part of the class
- the 'implements' part of the class
- an initialization block ({})
- a static initialization block (static{})
- instance variables with their access modifiers
- the added 'final' modifier for instance variables
- static variables with their access modifiers
- the added 'final' modifier for static variables
- constructors with their access modifier
- abstract methods and their access modifier
- instance methods with their access modifier
- an added 'final' modifier for instance methods 
- static methods with their access modifier
- an added 'final' method for static methods

I missed some modifiers (synchronized, native and strictfp) but furthermore I think it is near complete. To understand abstract classes and interfaces better, I'll cover these topics in relation to them.


### The access modifier of the class

Abstract classes have the same default access modifier as regular classes (package-private). An abstract class can be made public, but not private. You can set an abstract class to protected only if it is the subclass of another abstract class.

Interfaces have a package-private access modifier by default and you can make them public. Private and protected are not allowed.


### A 'final' modifier for the class

Abstract classes cannot have the final modifier, and neither do interfaces. It would defeat the purpose of both as their methods should be implemented or overridden.


### The 'extend' part

An abstract class can extend another class, either abstract or non-abstract.

An interface cannot extend a class.


### The 'implements' part

An abstract class can implement multiple interfaces. 

An interface can implement multiple interfaces but does so by using the 'extend' keyword instead of 'implements'. 


### Initialization block ({})

An abstract class can have an initialization block which is inherited by subclasses. 

Initialization blocks are not allowed in interfaces.


### Static initialization block

An abstract method can have a static initialization block which is inherited by subclasses.

Static initialization blocks are not allowed in interfaces.


### Instance variables with their access modifiers

Instance variables in abstract classes can have any access modifier. Default is package-private, as in regular classes.

Interfaces cannot have instance variables. If you define variables, they will be automatically set to public, static and final.


### 'final' in instance variables

Instance variables in abstract classes are the same as instance variables in regular classes and thus can be set to final.

Interface variables are by default public, static and final.


### Static variables and their access modifiers

Static variables in abstract classes work the same as static variables in regular classes. They can have any access modifier, default is package-private.

Static variables in interfaces are by default public and final. You don't have to declare either static, public or final.


### 'final' in static variables

Static variables can be made final in abstract classes. 

In interfaces static variables can only be final.


### Constructors and their access modifier

Abstract classes can have constructors with any access modifier. Default is package-private. They are inherited by their subclasses.

Interfaces can not have a constructor. Constructors are used to initialize object state, and an interface doesn't have object state.


### Abstract methods and their access modifier

In abstract classes, abstract methods are package-private by default and can be set to public or protected. Private is not allowed. Unlike in interfaces, you need to explicitly declare abstract methods abstract. Final is not allowed as modifier.

In interfaces, abstract methods are public by default and any other access modifier is not allowed. Final is not allowed either and 'abstract' can be omitted.

### Instance methods with their access modifiers

Abstract classes can have instance methods with any access modifier, be it private, public, protected or package-private. The latter is the default.

Interfaces can not have instance methods, only abstract methods, static methods and default methods.
CORRECTION: since Java 9 interfaces can have private instance methods, for the reason that they might be helpful when creating default methods. Private instance variable are not allowed.


### 'final' modifier for instance methods

Abstract classes can have the final modifier added to instance methods (but not to abstract methods)

Interfaces don't have instance methods.


### Static methods and their access modifier

Abstract methods can have static methods that can be called. Static methods are inherited by subclasses. Subclasses can not override these static methods, but they can shadow them with a new implementation. The static method of the superclass can still be called by using SuperClass.someStaticMethod(). Static methods in abstract classes can have any access modifier, default is package-private.

Interfaces can have public and private static methods. Private static methods can only be used within the class. The default access modifier is public.


### 'final' modifier for static methods

A static method in an abstract class can be set to final.

Interface methods can not be declared final, whether they are abstract, static or default. Static methods are allowed, they can be private or public, but declaring them final gives compilation error.


### Conclusion

There is no conclusion other than that you need to memorize some things. Yes, there is logic underneath the facts, but this logic won't provide answers to every question. Why, for example, can an abstract class be protected only if it inherits form another abstract class? Why can a static method in an interface not be set to final? 

Anyway, I hope to find some good heuristics to remember all this. Thanks to perplexity.ai who gave many of the answers.










