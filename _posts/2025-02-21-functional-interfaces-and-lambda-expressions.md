## Functional interfaces and lambda expressions

A new topic presented itself to me (yes, all in relation to 1Z0-819), namely functional interfaces and lambda expressions. I was not aware of functional interfaces at all and lambdas were a kind of unexplored territory.

In short, a functional interface is an interface that contains only one abstract method. Interfaces with just one abstract method can be used to create lambda expressions, which are function definitions with a very compact syntax. The community seems to love it, as it provides more elegant ways to do the same thing. The downside is some extra learning costs, a bit like learning a language with cases, like German.

Lambda expressions proceed where anonymous classes based on interfaces end. To understand lambda expressions it is best to first understand these anonymous classes. To create an anonymous class from an interface type you can do the following:

Let's say you created some generic interface like this (step 1):

```
interface SomeInterFace<J,K> {
    
    void someMethod();
    
    J someOtherMethod(K k);
}
```

To create an anonymous class based on this interface you do the following. The generic types are made specific and the abstract methods get their implementation (step 2):

```
SomeInterFace<Integer, String> myThing = new SomeInterFace<Integer, String>() {

    @Override
    public void someMethod() {
        // do something
        System.out.println("\uD83D\uDE03");
    }

    @Override
    public Integer someOtherMethod(String str) {
        // do something with a string and return an integer
        return str.length();
    }
};
```

With this, you can let the newly created object myThing do what you defined in the implementation (step 3):

```
myThing.someMethod();  // returns 😃
int myNameLength = myThing.someOtherMethod("Geert-Jan");  // returns 9
```

With functional interfaces, this process is being made less verbose. First of all, you do not have to create your basic interface. Java already made them for you in a lot of different variants like Function<K,V>, Supplier<K>, Predicate<R> and Consumer<U>. They are mostly generic and they have just one abstract method. They differ with respect to their arguments (number and type) and return type (or the absence thereof). 

These functional interfaces can be used instead of step 1, which saves time and reduces complexity.

In step 2, Java makes it possible to write the same thing but in a much shorter way. If you start your statement with this (I used the Consumer<K> functional interface here, a ready-made):

```
Consumer<Double> myMethod = 
```

Java actually already knows what you are gonna do. Yes, you can create an anonymous class similar to the previous step 2, but you are encouraged to omit every element that can be inferred by the compiler. 
- You do not have to again declare <new InterFaceName> with the used type as this is already mentioned before (Consumer<Double>). So you can start with the parentheses. 
- Between these parentheses you put the argument with the variable name. You can omit the type as you already declared it on the other side of the assignment. 
- Then you use the arrow operator (->) and then you can either create a block with curly brackets or just a single expression without brackets. 
- You can omit the name of the method (Java can infer it because functional interfaces have only one abstract method). 
- If you don't use the curly brackets you can also omit the return keyword, as the one expression you provide cannot be something else than the return value. 
- If there is no return value required (as in Consumer, which has 'void' in its signature), Java will know as well.

These are the two ways to do it, one without and one with a block.

```
Consumer<Double> myThing = (a) -> System.out.printf("Thanks for value %s!", a);
```

```
Consumer<Double> myThing = (a) -> {

    String s = String.format("Thanks for value %s!", a);
    System.out.println(s);
};
```

Right now myThing is an anonymous object of type Consumer that can use only one single method. The name of this method is accept(), as defined in the file where the Consumer interface is defined (java.util.function.Consumer.java). Other functional interfaces have different method names like test() and apply(). Below how you can use it.

```
myThing.accept(456.2);  // Thanks for value 456.2!
```

The magnificent thing about these functional interfaces and lambda expressions is that you can use them in streams. Streams filter and modify sequences of values of a same type and all the processing methods can be easily defined with lambda expressions. Streams are a big topic, more over them in a next blog.











