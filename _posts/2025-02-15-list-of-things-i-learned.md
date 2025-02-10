## List of things I learned

Repetition is key in learning so here is a list of what I learned while studying the first 90 questions in [this book](https://www.amazon.com/Oracle-Certified-Professional-Developer-Practice-ebook/dp/B08VRSQ3TW/ref=sr_1_1).

- variable names can start with an underscore but they need to contain more than only one underscore
- var++ returns the value of var before it is incremented, ++var returns the value of var after it has been incremented
- if you use var in type initiation, it will first try to make your value a primitive
- a string object cannot be reversed, it is immutable
- variables not declared as final can be labeled as 'effectively final'. Relevant with lambda expressions
- underscores in a numeric literal are allowed between digits but not adjacent to a decimal point
- if a variable in a method has the same name and type as an argument of that method, the variable in the method takes precedence
- the following code does not compile because you cannot retrieve the length of a Stringbuilder object when its construction is not finished in a chain of methods
```
var sb = new StringBuilder("radical").insert(sb.length(), "robots");
```
- ternary operators can be nested: x ? (a ? b : c) : d
- you cannot assign 'null' to a primitive 
- you can use byte, short, char, int, String, enumerated types and wrapper classes in a switch statement but not decimal primitives or non-string/primitive-wrapping Objects
- 'continue' in a while or for loop skips to the next iteration
- in a switch statement, you can omit the default or you can place the default somewhere else than at the end
- 0 and 1 cannot be used as false and true
- the value of a case statement must be a constant, a literal or a final value
- you can give loops names like V: (ahead of 'for' or 'do')
- for(;;) results in an endless loop
- while needs to be followed by an expression that is true or false, both in a while and a while/do loop
- you can omit brackets when there is only one statement to be executed. Switch can do without, while and do do not.
- beware of the fall-through principle in switch statements
- do- and while loops can be exitted early with a return statement
- do- loops require a body, a while loop doesn't
- both require a true/flase condition
- you cannot put multiple values in one case statement. But you can combine them like case 1: case 2:
- you can put 'return' in a switch statement but not in a switch expression
- a switch statement always needs braces, a while statement doesn't necessarily
- classes labeled 'final' cannot be extended
- static methods have no access to instance fields
- package-private is the default access modifier for classes, it is more restricted than protected because a protected class can be extended by a class in another package
- a default method in an interface can always be overridden by a class implementing the interface
- you can initiate a long with or without an L at the end. A float requires an F at the end, otherwise program won't compile
- new Parent().new Child() is a valid way of creating an instance of an inner class
- interface methods cannot be declared final. Static fields can, or actually they are by default.
- static interface methods can be declared private and public, but you can omit the latter as it is the default
- abstract methods in interfaces are public, you can make them private but then you need to give them a body
- anonymous classes have access to all members and variables of the enclosing class. If these variables are within the same method, they must be effectively final (value never changed after initialization)
- an anonymous class is an expression (part of a statement)
- you cannot apply .toString to primitives (toString is one of the methods within the Object class)
- a static main method cannot access instance variables
- a subclass cannot decrease the access level of an inherited method
- if you do 'Plant w1 = new Bush()', w1 will have the fields of plant and the methods of bush. 'Virtual method invocation only works for methods, not for instance variables.'

More to come!

