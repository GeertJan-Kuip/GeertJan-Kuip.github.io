# Definitions of higher level design concepts

I want to make tooling to analyze code. This brings me to the realm of what you might call 'higher-level' or 'abstract' design concepts (or patterns). Examples if it are 'dependency inversion,' 'static factory,' 'composition root,' 'domain object,' 'clear boundary' or 'interface segregation principle.' I want tooling that helps me to find these patterns in existing codebases.

## Prerequisites

The first prerequisite is creating a list of concepts/patterns that I want to be able to detect. The literature on software architecture can be helpful here but might not be exhaustive. I read just a limited number of books on architecture and design patterns, the most important being ['Design Patterns' by the Gang of Four](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8), the other one being ['Clean Architecture' by Robert C. Martin](https://www.amazon.com/dp/B075LRM681). Apart from what I found here, I have some things to add.

The second prerequisite in finding these patterns is creating unambiguous definitions, which is actually a difficult thing to do. For example, do the methods in a utility class (a concept) need to be abstract? Can a utility class have state, other than final constants? Must all methods be purely functional? Can a class that can be instantiated multiple times still be a utility class? If you create an interface and fill it with public static 'utility' methods, should you then call it a utility class?

## A list of concepts and patterns

I divide this section in three. The first is about concepts derived from 'Clean Architecture,' the second from 'Design Patterns' and the third from myself.

### Concepts from Clean Architecture

'Clean Architecture: A Craftsman's Guide to Software Structure and Design' is a well known book on OOP software architecture. Three key terms are 'boundaries', 'dependencies' and 'decoupling.' It defines a 'Dependency Rule' that says that _source code dependencies must point only inward, toward higher-level policies._ It advises you to define the most essential part of your application, or the part where 'inputs are converted to outputs,' and make this component independent. 

To achieve this, you should use dependency inversion, which practically means that the independent component should own the interface that is implemented by the class that is part of the component that is dependent on the higher-level-policy containing class. Being the owner of an interface that must be implemented by others makes you, literally, independent.

The book uses a range of acronyms to describe good practices. I summarize them below:

#### Single Responsibility Principle

A module should be responsible to one, and only one, actor. This principle is about functions and classes but at higher levels it will apply as well, albeit under different names.

#### Open-Closed Principle

A software artifact should be open for extension but closed for modification. Like the previous one, this principle applies on the level of methods and classes but also on higher levels.

#### Liskov Substitution Principle

This principle is from Barbara Liskov who wrote in 1988 that:

_If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T._

This is mainly a definition of polymorphism, as applied in the Java language. For Martin the real value of OOP is polymorphism, as it allows you to create clear boundaries between different parts of the software and to invert dependencies.

#### Interface Segregation Principle

There is a principle that software parts should not depend on things they do not need. The Interface Segregation Principle says that you should not depend on code that you don't need. Therefore it can be meaningful to create multiple interfaces for a part of software, so that different clients with different requests can all have their own limited interface.

Btw ChatGPT warned me not to let the number of interfaces explode because of this principle.

#### Dependency Inversion Principle

In the first place, this principle tells you that source code dependencies should refer to abstractions, not concretions. So the point of contact between two different parts must be an abstract class or interface, not a class containing implementations.

An exception can be made for very stable classes, like String in Java. It is safe to use the String class directly, there is no abstract class or interface required. But for more volatile elements, it is wise to have an abstraction inbetween. Stable abstractions are always better than volatile implementations. Some consequences mentioned by the book:

- Don't derive (inherit) from volatile classes
- Don't override concrete functions
- Never mention the name of anything concrete and volatile

Applying the principle leads, for example, to the creation of Abstract Factories that hide concrete implementations. Of course it is not possible to not mention the name of concrete classes, but this should be done in a separate part of the code, which can be the main function or some 'composition root'. 



- Interface Segregation Principle
- Dependency Inversion Principle

- Acyclic Dependencies
- Stable Dependencies Principle
- Stable Abstractions Principle





