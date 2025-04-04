## Scope, inner class and local class 

Paragraph [6.4](https://docs.oracle.com/javase/specs/jls/se11/html/jls-6.html#jls-6.4) of the Java SE 11 language specification deals with shadowing and obscuring. This blog narrows it down to the shadowing of variables by methods, the prohibition on the redeclaration of method variables within the method or within a lambda parameter within the method, and the properties of local classes.

### Example code

Below is very twisted code that compiles. Pointwise it contains the following:

- An inherited field named 'instanceString' hidden by a redeclaration in the subclass
- The field is shadowed in an inner class named 'Inner', just as a static field named staticString
- The field is shadowed in an inner static class named 'InnerStatic', just as a static field named staticString
- The field is shadowed in a method 'doSomething'
- The field is shadowed in a local class within a lambdabody
- The field is shadowed in a local class, also named 'Inner'
- The field is shadowed in another local class named 'OtherInner'
- The field is shadowed in another local class named 'Inner', which is nested inside 'OtherInner'
- The field is shadowed in the body of an anonymous class
- The field is shadowed in a local class named 'Inner2' in a try block
- The field is shadowed in a local class named 'CatchInnerClass' in a catch block
- The field is shadowed in a local class named LoopInnerClass in the block of a simple for loop
- The field is shadowed in a local class named SwitchInner in the block of a switch statement
- The method parameter 'a' is shadowed in local class Inner in the body of the lambda


```
class ShadowParent {
    String instanceString = "Instance variable";
    static String staticString = "Static variable";
}

public class Shadows extends ShadowParent{
    String instanceString = "Hidden instance variable";

    class Inner{
        String instanceString = "New one inside inner class";
        final static String staticString = "Nested static variable";
    }
    
    static class InnerStatic{
        String instanceString = "New one inside inner class";
        final static String staticString = "Nested static variable";        
    }

    void doSomething(Integer a){
        String instanceString = "Local variable";
        Consumer<Integer> myPrinter = (b) -> {
            class Inner{
                Integer a = 23;
                void doSomething(){System.out.println(a);};
            };
        };
        class Inner{
            String instanceString = "Variable nested in local class";
            Integer a = 10;
        }
        class OtherInner{
            String instanceString = "Variable nested in another local class";
            Integer a = 12;
            class Inner{
                String instanceString = "Variable double nested in local class";
                Integer a = 15;
            }
        }

        Serializable serializable = new Serializable() {
            String instanceString = "Nested in an anonymous class";
        };

        try{class Inner2{String instanceString = "Nested in an anonymous class";}}
        catch(Exception e){
            class CatchInnerClass{
                Exception e = new RuntimeException();
                String instanceString = "Nested in an anonymous class";
            }
        }

        for (int i=1;i<2;i++){
            class LoopInnerClass{
                String instanceString = "Nested in a loop";
            }
            System.out.println(new LoopInnerClass().instanceString);
        }

  
```

Every variable with the name instanceString is another variable. The reason I could put so many of them in this code is that inner classes and local classes created their own namespace. Few remarks:

- You cannot shadow method variables in blocks belonging to a for, a try, a catch, a finally, a switch or a lambda block. 
- If you want to do that anyway, the hack is to create a local class within these blocks
- The body of an anonymous class is shield from the method variables, which means you can do shadowing without having to create a local class

_Generalization: inner classes, static nested classes, local classes and anonymous classes add another level of shadowing to Java. You can redeclare method parameters and method variables in their bodies. This is not possible with for, try, catch, finally, switch or lambda blocks._

Furthermore: I tried to include a switch-case expression in this example as well (as opposed to a switch-case statement) but those were not allowed in Java 11.




