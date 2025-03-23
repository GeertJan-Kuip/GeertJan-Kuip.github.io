## Interface members - a story

Once upon a time in the 1990s, when Java was young, the makers of Java had given developers two types of members to use in their interface design. One was the public abstract method, the other was the public static final variable. Life was easy.

Because there were only two options to choose from, it was no problem to make all of the modifiers implicit. If you declared a method, it could not be anything else than abstract and public. If you declared a variable, it could not be different than public, static and final. Interfaces looked simple, no modifiers had to be used because the compiler already knew.

Times changed. A large body of skilled Java developers arose and they were hungry for a more complicated language with more ways to do the same thing with less lines of code. The makers of Java responded with two new additions to the interface. Starting with Java 8, it was possible to use static methods and default methods. It allowed for expressions like 'List.of(1,2,3)' and it didn't compromise the ideal of backward compatibility.

Unfortunately, it was not possible anymore to make all the modifiers implicit. From now on, every Java developer needed to use the 'default' modifier for default methods, and the 'static' modifier for static methods. The only modifier that kept its implicit status was the public modifier. Every member of an interface was public, and you didn't have to write it down explicitly.

This was not the end of the story. 'What if we allow developers to write private instance methods that they can use within their default methods,' said one of the Java makers. 'Yeah, that is a good idea,' said all the others. 'And what if we allow private static variables, that can be used within public static methods, or in default methods? That would be awesome too! The language has become too easy, too many people do Java, let's raise the bar! Think of how many ways this allows us to trick people in giving wrong answers on their Java exams!'

Of course these two new members of the Java interface could not have implicit modifiers anymore. They needed to be declared explicitly 'private', as was the case with the instance method, or 'private static', as was the case with the new variable. 

**CORRECTION:** 2025-03-23 I'm in the wrong about the addition of private static variables, they are not allowed. What I should have written was **private static methods**.

This is the reason that there are six interface members instead of two. This is also the reason that some members do not require access modifiers to be written down while others do.

