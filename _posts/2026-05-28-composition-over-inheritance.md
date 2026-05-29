# Composition over inheritance

For a while I have had a wrong understanding about the meaning of 'choose composition over inheritance'. I thought it meant that it was better to implement a set of interfaces than to extend an abstract class. 

Gemini just told me that composition actually means something different, namely including other classes, in their abstract form, as fields or method arguments in your class. You use them to delegate work to. I reread ['Design Patterns: Elements of Reusable Object-Oriented Software'](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/) and understand what Gemini meant.

Generally, I found out that other patterns as described in the book are a bit more sophisticated than I thought they were. There seem to exist patterns that are a sort of light-weight variant of the 'real' ones. The Factory Method in the book is more extensive than the static factory method that simply encapsulates object construction in a 'get' or 'instance' method. 

## The main principles behind the patterns

There are three main principles that guide the 23 patterns in the book:

- Program to an interface, not an implementation
- Favor object composition over inheritance
- Encapsulate what varies

This post focusses on the second point. The first point aligns with _Clean Architecture_, the third summarizes the structure of the patterns rather well. For 'encapsulate' you can as well read 'put in a separate class'.

## How to compose

I wondered what the definition of composition was, and in fact it includes every situation where there is referred within another object, either as field, as method parameter or (I suppose) as local variable. The encapsulating method can be name 

### Levels of delegation

Having said that, there are multiple levels of intensity. The most extreme is probably found in the Visitor pattern. Because of the use of keyword 'this', the process initiated in Visitor bounces between 

```
interface Behavior{
    
    String greet(String personsName); 
}

class RudeBehavior{
    String greet(String name){
        System.out.println("Hi " + name + ". You are a dickhead!");
    }
}

class PoliteBehavior{
    String greet(String name){
        System.out.println("Hello " + name ", nice to meet you!");
    }
}

class Person{


Design Patterns: Elements of Reusable Object-Oriented Software

