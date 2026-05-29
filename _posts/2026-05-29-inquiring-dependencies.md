# Inquiring dependencies

## A categorization

Dependencies can have several forms. They can be direct or they can be transitive, and furthermore they can have the following character:

- extends
- implements
- field (static or instance)
- method argument
- method return value
- local variable
- new operator
- cast operator

## Describing a dependency

To read code well I want to have a list of properties of dependencies that help me qualify the relationship between a class and a dependency. Below I discuss some aspects that might be relevant. In the end I want some algorithm that provides brief information on a dependency, in a way that I can understand the relationship between class and dependency in a glance.

### Abstract or not

If the dependency is an abstract class or an interface, there are a few things to investigate:

- Where does the coupling between compiletime and runtime type happen?
- Does coupling happen in some factory method?
- Are there multiple runtime implementations available/possible?

### Type of injection

Another question is how the dependency is injected. If it is a field, is it injected:

- via the constructor?
- via a dedicated setter?
- via member select within a method body/initializer not being a real setter?

If it is a method argument:

- Who calls this method? What is the call chain?

### Genesis

The next question is where the instance is created:

- In the field declaration itself? ```Vehicle car = new Car("Red");```
- In the constructor?
- In another class?
- In a method/initializer body?

### Level of delegation

In the previous blog post in which I discussed Gang of Four design patterns, I noticed there were several levels of delegation. The more the delegator and the delegate need to know about each other, the more extreme the delegation is. Visitor is an outspoken example of strong interconnection between delegator and delegate.

The question is, what objective indicators can you use to get an idea of the level of delegation?

- use of the 'this' keyword as argument to one of the delegate's methods (Visitor)
- argument propagation: a method on the delegate gets the argument of the enclosing method passed on (Strategy pattern)

### Methods being called

Method bodies might call methods that belong to dependencies. Of these method calls we want to know:

- What is the type of the arguments?
- Are there return values?
- How are these return values being used?

### Character of the dependency

Dependencies can be differentiated by the following aspects:

- Does it have transitive dependencies?
- How stable is this dependency (see the bog post 'Clean Architecture')
- Who else has this type of dependency?
- Who else has this specific instance of the dependency?

## How to use this

Ideally I want to have a tool that detects all sorts of patterns and relationships, faster than AI and more extensive than an IDE. But before I can create the appropriate algorithms, I need to have an idea what I am looking for.

Btw I'm planning to create two layers for analysis: layer 1 contains basics methods (findTransitiveDependencies, listMethodsCallingThisMethod etc). The other one, layer 2, creates more complex requests build from the basic methods in layer 1. The Command Pattern will help with this, I want to be able to create new analyses simply as subclasses of some AbstractAnalysis class (open-closed principle).



