# Composition over inheritance

For a while I have had a wrong understanding about the meaning of 'choose composition over inheritance'. I thought it meant that it was better to implement a set of interfaces than to extend an abstract class. 

Gemini just told me that composition actually means something different, namely including other classes, in their abstract form, as fields or method arguments in your class. You use them to delegate work to. I reread ['Design Patterns: Elements of Reusable Object-Oriented Software'](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/) and understand what Gemini meant.

Generally, I found out that other patterns as described in the book are a bit more sophisticated than I thought they were. There seem to exist patterns that are a sort of light-weight variant of the 'real' ones. For example, the Factory Method in the book is more extensive than the static factory method that I see often and that simply encapsulates object construction in a 'get' or 'instance' method. Same is true for the Builder pattern, the typical static inner Builder class found in code is often less smart than the Builder pattern in 'Design Patterns.'

## The main principles behind the patterns

There are three main principles that guide the 23 patterns in the book:

- Program to an interface, not an implementation
- Favor object composition over inheritance
- Encapsulate what varies

This post focusses on the second point. The first point aligns with _Clean Architecture_, the third summarizes the structure of the patterns rather well. For 'encapsulate' you can as well read 'put in a separate class'.

## How to compose

I wondered what the definition of composition was, and in fact it includes every situation where an object contains a reference to another object via its interface or superclass. This reference can have the form of a field, a method parameter or a local variable. 

'Design Patterns' uses the word 'delegation' to signify that a class can delegate operations to objects it has references. Two objects instead of one are now involved in the handling of a request. 'Delegator' is the name for the encapsulating object, 'delegate' is the name for the object inside this object.

### Levels of delegation

Having said that, there are multiple levels of intensity. The most extreme forms require the two classes to know of each other and the request handling is done by intensive coöperation. The most extreme is probably found in the Visitor pattern. Because of the use of keyword 'this', the process initiated in Visitor bounces between the two classes, one being Visitor and the other being the object that knows how to traverse some object tree.

A less extreme form of delegation is found in the Strategy pattern, where you have a field in your delegator holding a reference to an interface/abstract type that has one or more specific methods that are required to handle incoming requests. Instead of hardcoding the variants of this method(s) in the delegator itself, you delegate them to the delegate. As this delegate is referred to by its type/interface, you can decide which implementation to use. You can even swap the implementation for another implementation using a setter during runtime. Note how superior this is to inheritance, which cannot be altered during runtime. It is great for dependency inversion (delegator is not dependent on delegate because it is accessed by an interface that can be owned by the core of your application) and it is great with regards to the open-closed principle. You can extend your options by creating extra implementations of delegate in the outer layer of your program, without touching the business logic inside.

This strategy pattern forms a sort of middle ground, because the delegate still needs to know about the operations that delegator needs to perform. They are somewhat entangled.

The least extreme form is where delegator calls methods on delegate but where delegate doesn't need to know anything about delegator. If delegate is a list to which objects must be added, this list doesn't need to know about delegator, while delegator simply calls the 'add' method.

Summarized: extreme delegation differs from mild delegation in the way that delegator and delegate must know about each other. If 'this' is being used, we have an extreme form of delegation that is probably hard to understand (see Visitor pattern).

## Patterns using composition

Most patterns in 'Design Patterns' use some form of composition. There are two that do not:

- Template Method Pattern
- Factory Method Pattern

Both rely on subclassing/inheritance. Furthermore one variant of Adaptor (Class Adaptor) uses multiple inheritance, not composition.
