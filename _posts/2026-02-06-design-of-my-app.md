# Design of my app

The app that I'm working on, and that I want to have a good design especially with regards to the dependencies between the different packages that together make the application. The more business logic there is in a package, the less dependencies on other packages it should have, preferably zero.

Two principles will help with this ambition: dependency inversion and minimizing the use of custom domain objects.

## Dependency inversion

This topic is discussed in _Clean Architecture: A Craftsman's Guide to Software Structure and Design_ by Robert C. Martin. It is the fifth of the SOLID principles he discusses. This principle says that when component (a package) A depends on component B, but you want this dependency the other way around, you take the following steps:

- Identify the concrete class in B that A needs to import to function. Let's say this class is named 'Implementation'.
- Create an interface for that concrete class in component A. Let's say this interface has name 'ImplementationInterface'.
- Change the signature of Implementation to 'class Implementation implements ImplementationInterface'.
- Outside of package A, create an instance of Implementation with ImplementationInterface as reference tye.
- Pass this object to package A, using dependency injection or any other method.

The result is that class A does not have to know about the specifics of class Implementation anymore. Implementation is nowhere to be found in package A. A will access the Implementation object through interface ImplementationInterface, which is an interface that lives in A itself. A owns the interface and can now enforce the terms of the contract with package B, just by adjusting the interface. A is no longer dependent on B, while B has become dependent on A.

In this example I used an interface, it could have been done as well with an abstract class (although for this use, I think most developers would prefer an interface).

## Minimizing the use of domain objects

I recently listened a few of the lectures of Rich Hickey, the maker of Clojure. Clojure is a LISP-like functional programming langue that can be used on the JVM and interacts well with Java, meaning you can use its libraries. Hickey, himself a distinguished Java developer, complains about the tendency of Java code bases to include dozens of custom domain objects.

The downside of these typical domain objects is that you have to create a new one every time you solve a new problem. They are so specific that they can often not be reused, they increase the amount of boiler plate and they force the people who want to understand your code to learn the specifics of your domain objects over and over again.

Compare this to the more simple and basic objects like `List<T>` or `Map<P,R>`. These structures do not require extra code, especially when you do not fill them with custom objects, and they are so universal that you do not even be a Java developer to understand them. In functional programming lists are the central element and you can do almost everything with them.

With regards to the decoupling of components, it is beneficial to use basic data types form the standard library instead of custom domain objects. The dependencies that come with basic types are rock solid (java.util) and they are part of the vocabulaire of any Java developer. So generally: using basic types to pass around values between components is very often a good idea.

## My application

I'm talking about it already for months now and I am definitely progressing, especially with regards to my understanding of the Java compiler internal which I need to work with. 

The application will generate UML schema's of Java source code, code that is retrieved from a local filesystem or from a github code page. Being into architecture and thinking about future development, I have thought about the following three packages:

### Package 1 - retrieving files

The first package converts a user input (selection of a local directory, selection of a URL of a GitHub page) into a `Map<String, String>` of files. Some unique name based on package and filename is the key (I have to think about this) while the String content of the file (eventually stored as CharacterSequence) is the value.

The package can initially be small, but later on it can be extended to be able to retrieve code from other sources (GitLab) or to clone whole projects from GitHub or other Git-like platforms. Furthermore it requires code to detect and communicate all sorts of things that can go wrong. Think of invalid url, wrong filetypes, empty directories etc.

Choosing `Map<String, String>` as output value is in line with the minimalization of the use of custom domain objects. It would be logical indeed to create some JavaSourceFile object with fields for filename, content and more, and store them in some JavaSourceFileContainer, but that leaves you with more files, more code, and a less easy to understand interface between this package and the rest of the application.

### Package 2 - sorting out uml- and other data

This package utilizes java.compiler and jdk.compiler to sort out all the data required to make a somplete UML diagram. It recognizes packages and imports, it dissects classes and finds both the dependencies between classes and the dependencies of the package as a whole on outer packages. It stores the information in two types of custom domain objects, namely 'nodes' and 'edges'. 

These domain classes of nodes and edges will both have interfaces, and these interfaces will be stored in package 2 as well. Package 2 is independent, package 3 will depend on it.

### Package 3 -  
