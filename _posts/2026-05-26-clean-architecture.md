# Clean Architecture

I want to make tooling to analyze code. This brings me to the realm of what you might call 'higher-level' or 'abstract' design concepts (or patterns). Examples if it are 'dependency inversion,' 'static factory,' 'composition root,' 'domain object,' 'clear boundary' or 'interface segregation principle.' I want tooling that helps me to find these patterns in existing codebases.

In this post I want to summarize the principles I studied in ['Clean Architecture: A Craftsman's Guide to Software Structure and Design' by Robert C. Martin](https://www.amazon.com/dp/B075LRM681). In subsequent blogs I want to discuss how these principles can be properly defined, so that I can write algorithms that help me to detect them.

## Concepts from 'Clean Architecture'

'Clean Architecture: A Craftsman's Guide to Software Structure and Design' is a well known book on OOP software architecture. Three key terms are 'boundaries', 'dependencies' and 'decoupling.' It defines a 'Dependency Rule' that says that _source code dependencies must point only inward, toward higher-level policies._ It advises you to define the most essential part of your application, or the part where 'inputs are converted to outputs,' and make this component independent. 

To achieve this, you should use dependency inversion, which practically means that the independent component should own the interface that is implemented by the class that is part of the component that is dependent on the higher-level-policy containing class. Being the owner of an interface that must be implemented by others makes you, literally, independent.

### Concepts from Clean Architecture - Modules, Packages, Classes

The first part of the book describes five principles that should be followed when you organize classes, packages and modules:

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

### Concepts from Clean Architecture - Component Cohesion

The concepts below have 'Component Cohesion' as overarching theme. The author discusses how you should decide which classes belong to the same component, whereby component is defined as a unit that can be independently deployed. It can be an application or a library, and I would say that you can define it as a set of classes that when you compile them, no symbol definitions will be missing.

#### The Reuse/Release Equivalence Principle

Classes and modules that are grouped together into a component should be releasable together. They share the same version number and use the same release documentation.

#### The Common Closure Principle

This is the defintion in the book:

_Gather into components those classes that change for the same reasons and at the same times. Separate into different components those classes that change at different times and for different reasons._

Note how similar this is with the previously discussed Single Responsibility Principle.

#### The Common Reuse Principle

_Don't force users of a component to depend on things they don't need_, is what the book definition says. Note how this aligns with the Interface Segregation Principle. If a component is only used partially by its clients, one should consider to split it up.

### Concepts from Clean Architecture - Component Relations

The following three principles deal with the relationships between components.

#### Acyclic Dependencies

This principle states that there should be no cycles in the dejpendency graph. If A depends on B, B should not depend on A, neither directly or indirectly. When there are multiple points of contact, all arrows between the components must point in the same direction.

A direct problem of cyclic dependencies on the level of components is that you cannot build the application anymore. Normally you start with the independent module and then follow the graph, but this becomes impossible when cyclic dependencies exist.

Note: on the lower level of classes and packages cyclic dependencies are not a problem in a technical sense, although you should consider if you can do without. In the jdk.compiler library classes Symbol and Type have each other as dependency which is okay.

Note: to break a cycle, you can use dependency inversion.

#### Stable Dependencies Principle

This is actually a very interesting principle. It distinguishes between stable and unstable components, saying that any component must only have dependencies more stable than itself. 

Stability (or actualy Instability) of a component can be calculated useing the following procedure:

- Count the number of classes outside this component that depend on classes within this component (Fan-in)
- Count the number of classes inside this component that depend on classes outside this component (Fan-out)
- Instability now is Fan-out / (Fan-in + Fan-out)

This metric is interesting (I object to the fact that transitive dependencies are not inncluded) and for example tells you that String is hyperstable and the composition root in your application, where the program starts, has maximum instability as nothing depends on it.

Author notes that once you see that some component depends on a component less stable than itself, you should consider using dependency inversion.

#### Stable Abstractions Principle

If, according to all principles discussed, the most important part of you code ('policy') resides in the most stable component, you might note that it becomes impossible to change the code within this policy component because all other components will break if you do so.

The solution is the Stable Abstractions Principle, that says that you should hide the implementation of you high-level policy behind abstractions, within the same class. This can be deduced from earlier principles as well. 

To measure the abstractness of a component, author provides a new 'A' metric:

- Nc: the number of classes in a component
- Na: the number of abstract classes and interfaces in a component
- A: Abstractness A = Na/Nc

Components with high abstractness (A) should be stable (low I) and vice versa. The book shows a twodimensional diagram with A and I on the axes and states that component should reside near the line A+I=1. Author calls this line the _Main Sequence_. It is worth reading this specific subchapter of the book, as it combines two specific metrics to create a new sort of information about a component.

If I must summarize it, it says that a stable component should be abstract, while an unstable component (having many dependencies without being a dependency for other components) should not be abstract (meaning it has no need for interfaces and abstract classes to communicate with the other components).



