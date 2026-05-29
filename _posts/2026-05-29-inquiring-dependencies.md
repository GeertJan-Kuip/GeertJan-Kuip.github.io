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

## What else you want to know

To read code well I want to have a list of properties of dependencies that help me qualify the relationship between a class and a dependency. Below I discuss some aspects that might be relevant. In the end I want some algorithm that provides brief information on a dependency, in a way that I can understand the relationship between class and dependency in a glance.

### Level of delegation

In the previous blog post in which I discussed Gang of Four design patterns, I noticed there were several levels of delegation. The more the delegator and the delegate need to know about each other, the more extreme the delegation is. Visitor is an outspoken example of strong interconnection between delegator and delegate.

The question is, what objective indicators can you use to get an idea of the level of delegation? I summed up a few below.

### Abstract or not

If the dependency is an abstract class or an interface, there are a few things to investigate:

- Where does the coupling between compiletime and runtime type happen?
- Are there multiple runtime implementations available/possible?

### Type of injection

Another question is how the dependency is injected. If it is a field, is it injected:

- via the constructor?
- via a setter?
- via member select within a method body/initializer?

### Genesis

The next question is where the instance is created:

- In the field declaration itself? ```Vehicle car = new Car("Red");```
- In the constructor?
- In another class?
- In a method/initializer body?

### 

If it is a method argument:

- who calls this method?

If it is a local variable:

- is it the return value of some other method?
- is it created via 'new'?
- is it 


- If methods are being called on this type, is 'this' then passed as argument?
- If a method is called on this type, is it then provided with an argument that is an argument 