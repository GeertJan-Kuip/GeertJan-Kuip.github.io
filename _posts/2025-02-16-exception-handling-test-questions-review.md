## Exception handling - test questions review

In my [book with exercises](https://www.amazon.com/Oracle-Certified-Professional-Developer-Practice-ebook/dp/B08VRSQ3TW/ref=sr_1_1) I tested myself with 40 questions I found in chapter 4 (questions 7-46) about exception handling. Of those 40 questions, I answered 22 correct which gives me a score of 55%. I need 68% and therefore I would need at least 5 more correct answers. I reviewed my result as follows:

- Of the 24 questions that included sample code, I only got 10 correct. 
- Of the 16 questions without code samples, I got 12 correct. The theoretical part is easier for me.

### In detail
Multiple times I had problems identifying hierarchical relations between classes as the key to the right answer. Examples:
- When a catch statement has two or more exception types in it, code will not compile if one is a subclass of the other
- If two custom exception classes are created, one a subclass of the other, and the subclass creates a constructor using super but with another type of argument than the parent, compilation fails.
- One question (7) presented a custom exceptionclass extending Exception. It had three constructors, two with super and one with an empty body. All three were valid because the constructors were analogous to those of Exception.
- One question (39) required me to know that UnsupprtedOperationException is a subclass of RunTimeException.
- One question had code in which a catch statement was not able to catch the exception because the exception type thrown was higher in the hierarchy than the exceptiontype in the catch statement.

There were two questions in which code would not compile because a variable was declared multiple times. 
- In question 13 it was a 'v' that was declared as an argument in main and later as the name for a RunTimeException object 
- In question 38 it was an 'e' that was reused within the same function with declarations for different (custom) exception types.

A mistake I made twice was not seeing that exceptions that are thrown by a method but not by the caller (and not caught either) result in compilation error. 

There were two questions were thrown exceptions were not caught. I had the answers right except that I forgot that the output of the printed stacktrace (unhandled errors give stacktraces in the terminal) also was part of the output of the code.
- Question 8 had a try-catch-catch-finally sequence. The first catch statement caught an exception but threw another one that could not be handled. The output was that lines printed fromout the try, the first catch- and the finally block were printed to the terminal, and only after that the stacktrace was printed. Finally has precedence.
- In question 37 the program lead to 4 regular printed numbers, indicating smooth program flow, and then a printed stacktrace because of an unhandled exceptionn. The printed stacktrace came at the end as the finally blocks got first.

Question 28 was not about exceptions, it had this code and I was not aware that this is a valid way to create an object. AutoCloseable is an interface, but because you make it an anonymous class it works.
```
var weatherTracker = new AutoCloseable(){
	public void close() throws RunTimeException {
		System.out.println("Thunder");
	}
};
```


Lastly, four trick questions:
- try-with-resources statement put between curly braces instead of parentheses. I didn't notice.
- creation of two custom exception classes that did not extend Exception or any other class. I overlooked it.
- semicolon at the end of the last statement within the try-with-resources block, just before the closing parenthesis. I was not sure if this was okay.
- Is this declaration valid? Even though it is a very misleading name for a class, the answer is yes.
```
class Error extends Exception {}
```

### To do
I think I must do the following:
- always be aware of the hierarchical relationships between classes 
- learn exceptiontypes and remember whether they are subclasses of RunTimeException or not
- fiddle around in Intellij and create my own custom Exceptions, relying on constructors of parent classes with the super keyword
- remember that unhandled exceptions or exceptions thrown by main to its non-existing caller result in terminal output
- watch out for duplicate variable names and double declarations
- remember that the output of the finally block preceeds the stacktrace printing to the terminal that is the result of uncaught exceptions
- experiment with creating objects using anonymous classes





