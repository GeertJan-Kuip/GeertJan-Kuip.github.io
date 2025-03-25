## Inner classes can do anything

There are four types of nested classes, namely:
- Inner class
- Static nested class
- Local class
- Anonymous class

This post is about inner class. Just like enums, inner classes allow almost everything that regular classes allow. The possibilities are thus far more than one would typically need, especially since Java 16 when the ban on static methods, static initializers and static fields was lifted. Before Java 16, you could use final static fields but not more. Since Java 16, even rather stupid things are possible, such as an inner class extending the enclosing class, or an inner class implementing the same interface as the enclosing class. 

### Perplexity is still confused

Perplexity is great but when I asked to list the things that regular classes could do and inner classes couldn't, I got a list that is largely outdated now. The following differences do not apply anymore:

- Inner classes cannot have static fields or methods (they can)
- Inner classes cannot contain static initializer blocks (they can)
- Inner classes cannot be declared with top-level access modifiers like public or package-private, as they are always members of their enclosing class (they can)
- An inner class cannot extend its enclosing class, as this would create a circular dependency (I tried and it was allowed)
- Inner class cannot implement the same interface as the enclosing class (it is allowed now)
- Inner classes cannot have a public static void main(String[] args) method (yes they can, although you might not be able to use them as startpoint for the program)
- Inner classes require an instance of the outer class to access its static members (not true anymore, proof in code below)

The differences that remain are the following:

- You cannot create an instance of an inner class without first creating an instance of the outer class
- Inner classes require an instance of the outer class to access its static members

### Problem on the 1Z0-819 test

The upcoming test takes Java 11 as norm. This means that I have to step back in time and do as if static elements are not allowed, except for static final fields, and that I cannot implement the same interface as the enclosing class or extend the enclosing class.

### Maxing out

I wrote some code to see what you can do with inner classes in Java:

```
public class Outer implements Serializable{

    static String s = "An instance method was called";

    public void instanceMethod(){
        System.out.println("This one has the same name as a method in the inner class. No problem.");
    }

    protected class Inner extends InnerAbstract implements Serializable {  // this extension and implement were not allowed before 
        static {System.out.println("Static initializer activated");}  // was not allowed before
        {System.out.println("Instance initializer activated");}

        static void staticMethod(){				// also not allowed before
            System.out.println("Static method called");
        }

        @Override
        public void instanceMethod() {
            System.out.println(s);
        }
    }

    abstract class InnerAbstract extends Outer{
        abstract public void instanceMethod();
    };
}
public static void main(String[] args) {
    Outer.Inner.staticMethod();  // this is not in the spirit of the classical inner class but you can do it now.
    new Outer().new Inner().instanceMethod();
}

//output:
"Static initializer activated"
"Static method called"
"Instance initializer activated"
"An instance method was called"
```

Given the restrictions that existed in earlier versions this is quite remarkable. You can create a rather complex microcosmos within a class, with abstract classes, extensions, implementations etc. The line ```Outer.Inner.staticMethod();``` even shows that static methods within an inner class can be called without ever creating an instance of the outer class, something that was typically reserved for static nested classes.

And inner classes can have the private and protected access modifier, which is not allowed for top level classes. You might say that inner classes provide almost more freedom than regular classes.

### About namespaces

Fields and methods in enclosing- and nested classes are not in each others namespace, even though inner classes can access members of the outer class freely. The rule is that inner classes can call members of the enclosing class directly, but not vice-versa, and that if the inner class has a member with the same name, it will select this one automatically unless you work with ```Outer.this```. Example:

```
public class Outer {

    String s = "Outer";

    protected class Inner {
        String s = "Inner";

        protected void printSomething() {
            System.out.println(s);
            System.out.println(Outer.this.s);
        }
    }
}

public static void main(String[] args) throws Exception {
    new Outer().new Inner().printSomething();
}

// output:
"Inner"
"Outer"
```

Note the use of ```Outer.this.s``` to indicate the s variable of the Outer class. ```this``` explicitly refers to instance context, without ```this``` it would not be able to access an instance variable but it would work to access a static variable. The test can include questions with deeply nested classes where you will have to use this.varName, Enclosing.this.varName, EnclosingEnclosing.this.varName etc. Chapter 12 of the book has a code example.




