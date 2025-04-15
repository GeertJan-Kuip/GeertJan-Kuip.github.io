## Native modifier and clone()

While working on chapter 22 of the book about security and the use of copy constructors I encountered on something I didn't grasp at first, namely the way the clone() method of Object is implemented and inherited. Step by step:

- Object has an abstract method named clone() that throws a CloneNotSupportedException.
- When using clone on an instance object that does not implement interface Cloneable, the CloneNotSupportedException is thrown during runtime.
- When you do implement interface Cloneable without implementing the method, you are able to create shallow copies without problems.
- This is weird, because Object only has an abstract method clone() and Cloneable only has an empty-bodied method clone.

All in all, no implementation of clone() is found in either Object or Cloneable and nevertheless the clone() method works without a custom implementation. 

I asked Perplexity how this was possible and it came with the answer:

**Native Implementation in Object**
_The Object class contains a native clone() method that performs a bitwise shallow copy of the object's fields. This implementation is hidden at the JVM level._

**Role of Cloneable**
_The Cloneable interface acts as a marker to grant permission for cloning. If a class implements Cloneable, the JVM allows the inherited Object.clone() method to execute without throwing CloneNotSupportedException._

The hint I should have seen is in the method declaration of Object's clone() method:

```
protected native Object clone() throws CloneNotSupportedException;
```

It contains the native modifier, who's definition is the following according to the [Java 11 language specification](https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.4.3.4):

_A method that is native is implemented in platform-dependent code, typically written in another programming language such as C. The body of a native method is given as a semicolon only, indicating that the implementation is omitted, instead of a block (§8.4.7)._

So this is where the typical behavior of the clone() method comes from. The shallow copy you get when implementing Cloneable is created under the hood, maybe in C.  

Btw implementing Cloneable is not mandatory, you can provide a custom implementation of clone() without implementing Cloneable and as far as I could see this works just fine. 