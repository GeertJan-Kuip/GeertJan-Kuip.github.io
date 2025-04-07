## What I see and what the compiler sees

Chapter 12, review question 11 of the book taught me that the Java compiler does not have a theory of mind and that I need to be aware of it.

The question presented a code snippet and asked that if there was an error in the code, to what line the compiler would point. This is the snippet:

```
public interface CanWalk{
    default void walk() { System.out.print("Walking");}
    private void testWalk() {}
}
public interface CanRun{
    abstract public void run();    
    private void testWalk() {}
    default void walk() { System.out.print("Running");}
}
public interface CanSprint extends CanWalk, CanRun {  // line m1
    void sprint();
    default void walk(int speed){ System.out.print("Sprinting");}  // line m2
    private void testWalk(){}
}
```

The main thing happening here is that CanSprint inherits the default method 'walk' from both parent interfaces. To resolve the ambiguity, one has to implement this method, using the correct signature. CanSprint implements the walk method, but with the wrong signature. It has one int parameter while the inherited method(s) have zero parameters. This is clearly wrong.

Okay, about theory of mind. My automatic thought was that the programmer had made a mistake in line m2, accidentally using the wrong signature. So my answer was that the compiler would point to this line m2.

But how can a compiler know that the programmer made a mistake? The programmer might have very well wanted to create a new signature and just have forgotten to override the inherited walk method. The compiler doesn't know what the programmer thinks, and only knows that interface CanSprint does not have an override of walk(). That leads to a problem in line m1, of the interface declaration and that was the right answer.

You might say it is a trick question (there were more possible answers, one being that the code would compile just fine). Nevertheless, it is much better to learn to think as a compiler.





 