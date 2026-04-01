# How to understand code fast

The app that generates uml schema's from source files (.java) is meant to understand Java code easier. Instead of displaying all information on the code, as Intellij or any other IDE does, it should be very selective and only display three (plus or minus two) pieces of information at the same time. 

Those pieces of information should be highly effective in increasing your understanding of dependencies, module- and package design, the character and role of individual classes, inheritance relationships, program flow, instance creation etc. 

## Dependencies and centrality

UML diagrams are a sort of default way of visualizing dependencies. The problem with displaying uml is that it is just too much information. Instead I can do the following:

### Provide the names of the top n most central classes/packages/modules

I introduce the term 'centrality' here, which correlates with the number of (transititve) dependencies a class has on classes within the same package and/or within the same module and/or within the same set of source files. I suspect that a certain type of package has a structure whereby some classes are independent, some have a few dependencies, and some have many. 

An example of a class where everything comes together is com.sun.tools.javac.main.JavaCompiler, which is the central class in the compilation process. There is, I suppose, a path from every class involved in the compilation process to JavaCompiler. A class less central would be com.sun.tools.javac.code.Type. 

Instead of using 'class' as a unit to analyze centrality, you can also do the same analysis on package- or module level.

### Provide the names of the (almost) independent classes/packages/modules

Certain classes, packages or modules live on the edge of the graph, having few dependencies on other classes within the same project. This is true for a class like com.sun.tools.javac.code.Kinds, or even more for interfaces like com.sun.tools.javac.code.Formattable and com.sun.tools.javac.code.Messages. 

Having the list of those classes that are more or less independent is useful, as it reminds you that for the program structure and flow you often can gloss over them.

### Create and indented list

To see how a central object (like JavaCompiler) wraps classes further from the centre, you can create a sort of tree diagram that shows the class on hand as root and that branches out to all transitive dependencies. Of course many transitive dependencies will be a dependency of more than one primary dependency, and actually the underlying structure is a graph and not a tree, so either I need to accept doubles in transitive dependencies or I need to omit doubles. Nevertheless, by drawing it as a tree it is easier to see how the class on hand relates to other classes, packages and modules.

### Find the number of dependent classes

Some classes might combine low centrality with a lot of classes that depend on them. It is interesting to know which classes have the most classes that depend on them. They might be the typical carriers of important data, like Type, or they are a useful utility class. 

## Studying individual classes

There are a few metrics that might help to understand the role of a class within the larger application. I thought of the following:

### Instance creation

It seems to be super useful to know where and how class instances are being created. Where, for example, are Tree/JCTree objects being born? Scanning all method and initializer blocks, plus the field declarations, for New statements is the way to go. In case of instance creation via factories and builders it becomes relevant to know which methods call these indirect constructors.

### Amount of instances created

Apart from knowing where the new instances are being born, it is good to know whether there will be multiple instances (created in some loop, stream or recursive method) or that there is only one instance being created. In case of a utility class there might even be zero instances being created.

There is a fundamental difference in classes with 0, 1 or many instances, they play fundamentally different roles. Javac creates tons of Tree, TreePath, Symbol, Type, Name and other objects, but only one JavaCompiler object. That is important to know.

### Interfaces and superclasses

Knowing under what compile-time object types the class can exist is interesting, so at least you can recognize the class if it appears under the banner of an interface or superclass. To do well in this respect, it is good to scan the code for lines where the new instances are being generated.

In this respect it might be smart to differentiate between the set of all possible polymorphic types for a certain type, and the set of realized polymorphic types. It is well possible that even though a certain runtime object can exist under 5 different compile-time types, only one of them is ever being applied. 

### Top 3 method return types

A class can have many methods, scanning the list of all signatures results in information overload. Instead you can make an overview of the return types of the methods, excluding void, and rank them. Knowing this will give you some idea of what this class is all about.

Example: `com.sun.tools.javac.code.Scope` has `Symbol` or `Iterable<Symbol>` as most frequent non-void return values. This aligns with the role of Scope, which is a class that keeps track of all symbols in a namespace.

In the subclass `com.sun.tools.javac.code.Scope.ScopeImpl` you find `Symbol` and `Iterable<Symbol>` as well, but `WritableScope` and `Entry` occur more frequently as return values. This makes you wonder what these classes are, and why they are relevant. `WritableScope` is just a superclass, while `Entry` is something rather sophisticated that improves the performance of Scope lookup, especially finding sequences of shadowed variables.

### Javadoc comments

Simply reading the javadoc comments of a file helps. The app might be able to tell you whether there are comments in teh file and if so, show them, without the code itself. It is simple reading, and you should learn something from it.

## Methods

### Applying AI

Understanding the intricacies of (sophisticated) methods is difficult and I'm helped a lot by ChatGPT for this. Being able to connect to some AI-provider API and feed it (part of) the code, so it can explain a thing or two about a specific method, would be rather helpful. 

### Call stacks

Tracing method execution, jumping from method to method, can very well be done with the Intellij or any other debugger, and while it works perfect, it provides way too much information. Having a simple overview of a call stack, starting at method A, would be great. Creating the correct graph of method calls is nevertheless state dependent and troubled by overrides. I have not decided on some approach here.

## Hard problems

### Compile-time type vs runtime-type

The hard problem in analyzing code is about tracing method calls. This is problematic because of overrides, Java selects the method version based on the runtime type and not the compile time type. What you would love to know is the runtime type of any symbol, but `variable.type.tsym` only provides the compile-time type. 

Based on a [Youtube video](https://www.youtube.com/watch?v=dPLaVxVuPd8) on the Java channel, by Christian Wimmer who works on the GraalVM project, and on a chat with ChatGPT, I came to the conclusion that finding the runtime types and more generally constructing the method call stacks from static code analysis is pretty hard or almost impossible. Apart from the problem of runtime types, there are also all the conditionals. I might need some simple hack for this.

### Annotations

I have no experience with annotations other than Spring. Assuming that static code analysis of a Spring project might be a completely different thing to do, I have yet no clue how to do it. Retrieving annotations from the AST is no problem, so maybe Spring projects can be analyzed in a similar but otherwise totally different way. I have no clue how to get info on dependencies, centrality etc, maybe it is just enough to know the Spring annotations to know the role of each class.

When I tried to understand the Maven codebase, I encountered something with compile-time annotations that resulted in the creation of new classes during the compilation process. I read about how this works, but do not know yet how to deal with it in this project.

## Some ideas

### A note-taking system

In .java files you can write comments that might help you or someone else later one. You can even create those typical html pages from them to document your project.

While I cannot alter the files I'm studying by writing in them (at least I do not want to) I was thinking of the possibility of creating notes on program elements, that can be stored in some file. It is a sort of simple annotation system that lets you take notes while studying the code, whereby each note belongs to a certain class, method, package or whatever.




