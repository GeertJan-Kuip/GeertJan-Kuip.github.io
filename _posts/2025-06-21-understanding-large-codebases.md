# Understanding large codebases

Over the past two weeks I have been busy trying to understand the Maven code base. It is big and although I learn a lot, I have not yet found a strategy that allows me to sit down, follow a procedure and end up with a full understanding of the code.

## Things that are difficult

#### Obtaining a mental map of inheritance trees 

Many classes in Maven, and in other code as well, have children and/or a parent class. Naming gives a clue, like in BaseParser being the parent of MavenParser or LookupContext being the parent of MavenContext. Nevertheless, this inheritance forces you to look in more than one class and to remember more than one name. And it is confusing when trying to learn the instance fields, because they can be in either class.

#### Reference type vs object type

It is a great thing that Java distinguishes between reference type and object type but when trying to grasp the code it is confusing. You might see a method invocation in the code and see that it returns a type Parser (the reference type) but you need to go to the implementation to see what the actual type of the newly created object is. If you are lucky you can find it in one step but it can take more than one.

#### Single responsibility

The accepted design philosophy is that every class, interface, enum or record must have a single task or responsibility. One object collects the data, a second stores it, a third does calculations with it. All fine but you end up with a lot of classes. The maven-core code has more than 900 .java files. Okay, you can skip the Exception classes from your mental map, and because of inheritance and polymorhism the number of 'real' objects is less, it is still an awful lot.

#### Instance fields initialized with null

To know what a class represents it is useful to not only look at the name, the package and the javadoc but also the instance fields. If the instance fields are int width, int height and int length the class probably represents a box. What is annoying is that not every field is set at the same moment and sometimes you really do not know if some instance fields will just keep their null value forever. In the Maven project I noticed that values that will be set are annotated with @Nonnull or are wrapped in a requireNonNull() method, so you can infer that all instance fields not having this annotation or wrapper will probably be set at a later time.

#### Nested objects

A complex object within a complex object makes it a bit harder to trace where all the values are. In maven-cli, where the main entry point is, a ParserRequest object is created that contains things like the args from the main method, some strings and paths, but also more complex things like a Lookup object (whose object type is one of the implementations of Lookup), a MesageBuilderFactory object and a Logger object. While their names indicates what they are about, you still can only guess. But now comes the real thing: this ParserRequest object ends up as a value within an InvokerRequest object, which actually has BaseInvokerRequest as object type. The nesting thus becomes rather complex and it requires a lot of work to find out what exactly is in the InvokerRequest object. 

#### Names are never clear

Names in Java are supposed to be as clear as possible. That is good, but it doesn't mean you can automatically understand what a name really means. BaseInvokerRequest has an instance field TopDirectory of type Path. It took me quite a while to find out that TopDirectory indicated the path of the parent module in a multimodular project. It could have meant something else as well.

## The crux

Given all the intricacies of studying large codebase, I have a few thougths:

### 1 - Just make someone responsible

In an interview with either Lex Friedman or ThePrimeTimeagen Chris Lattner, who developed the Swift language while working at Apple, told that most senior developers in the software world derive their status and utility from the fact that they know a large chunk of code really well. They act like guardians over a code base of which they know about everything. Given the complexity of large codebases and the time it takes to get familiar with it, this seams a reasonable solution to me. Just like a manager of a supermarket knows all the ins and outs of his specific shop or a car mechanic knows all specific car models or components really well.

### 2 - Solution 1 is not ideal

It means you have given up (I exaggerate) on better ways to understand code in a faster way. 

### 3 - The search is not the problem

Finding out how code works, I suspect, is not the most difficult part of it all. Yes, it takes a lot of time to go through the code, finding out what every field and object really represents etcetera. But basically you can just start and work your way through it.

### 4 - It is the mental representation

If you have found out how some part of the code works, how do you write that down? Do you create text, meaning that you create some extra layer of javadoc-like documentation? Will you be writing down schema's? Uml has not been a success, somehow schema's do not really grasp the whole essence. There is just a ton of detail in the code and knowing those details is actually important. Somehow you must know what the creator of the term "TopDirectory" meant.

## Closing statement

I found out that while figuring out the code base of Maven, I had no good way of translating my new pieces of knowledge into documentation that I would be able to read a week later and still understand what it was all about. Texts I made became as complex as the code itself, schema's left out too many details.

This is what I want: a convenient way to document my explorations in the code in a way that this documentation makes actually sense at a later point in time.







