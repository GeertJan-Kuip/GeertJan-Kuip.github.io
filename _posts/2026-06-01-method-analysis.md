# Method analysis

With Symbol, the forbidden-access class in jdk.compiler found as com.sun.tools.javac.code.Symbol, you can get some information about methods by using the methods and fields belonging to either subclass MethodSymbol or to Symbol itself.

In this blog post I will provide a list of methods and fields from class MethodSymbol (including its abstract parent Symbol) that are directly available, whereby the `flags_field` field of Symbol is important. After that I will provide a set of other pieces of information about methods that I want to generate.

## Symbol - fields and methods

Symbol is an abstract class and parent of MethodSymbol.

### Fields

Symbol has only seven fields which makes it comprehensible. Those fields are:

#### Kind kind

Kind is an enum from the same package as Symbol. It lives in the Kinds class, the Kinds class being a sort of wrapper in which another enum (KindName) and a stat ic inner class (KindSelector) live as well.

Kind tells you what sort of Symbol you are dealing with, note that because superclass Symbol itself is abstract it cannot be a runtime type itself. Furthermore it has values that probably have to do with the compilation process, in which intermediate states might require special symbols or in which errors are being found.

My take on it: irrelevant for my purposes.

#### long flags_field

This is the interesting one. Every symbol has a 64 bit long attached to it and every one of these 64 bits represents some true/false property. Many of these flags have to do with compiler processes so are not relevant for my usage, but there is a considerable list of flags that I might want to use. A few examples:

|Bit|Name|Description|
|---|---|---|
|1-11|PUBLIC, PRIVATE, FINAL etc|standard Java/modifier flags|
|33|PARAMETER|Flag for VarSymbol that is a method parameter|
|34|VARARGS|Flag that marks method having varargs parameter|
|43|DEFAULT|Marks a default method or an interface containing default methods|
|51|MODULE|Flag to indicate class symbol is for module-info|
|52|AUTOMATIC_MODULE|Flag to indicate the given ModuleSymbol is an automatic module|
|17|BODY_ONLY_FINALIZE|Flag that marks finalize block as body-only|
|61|RECORD|Class is a record|

Note: to many of these flags there is a method that returns whether the flag, or multiple flags, are set. If there is an appropriate method available, you do not have to access the flags_field field yourself.

Note: there is an enum ElementKind in the accessible api (javax.lang.model.element.ElementKind). This enum value can be requested using all sorts of `getKind()` methods. ElementKind does not go as deep into compiler processes as Flags but tells you correctly what sort of Symbol you are dealing with, including the differences between different ClassSymbols (CLASS, RECORD, INTERFACE, ENUM) or between different VarSymbols (FIELD, LOCAL_VARIABLE, PARAMETER etc).

#### Name name

The name of the Symbol stored as a Name object. Name has a more efficiënt way of storing char sequences than String. It basically creates one long char array, adding every new name at the end and returning the start and end position of it.

#### Type type

Every Symbol object has a corresponding Type object and vice-versa. This is not completely true, a Symbol object can have zero or more associated types. PackageSymbol has no associated Type for example, while Name can have multiple Type associated with, for example because `Name` and `Name[]` are stored as different Type objects.

#### Symbol owner

The parent of this symbol. If you want to know in which class the symbol lives, use method `enclClass()` or (for inner classes) `outermostClass()`. These methods recurse upward until they meet a non-nested ClassSymbol.

#### Completer completer

This has to do with the 'lazy' aspect of compilation. When a Symbol or its Type has not fully been resolved, the `complete()` method on its Completer can be called (at least I think so). 

#### Type erasure_field 

Documentation says: "A cache for the type erasure of this symbol."

### Methods

There are 57 methods (Java 21) so I have to be selective in what I discuss here. Only methods relevant for MethodSymbol are included. I put them in a table for brevity.

|Method|Use|
|---|---|
|isStatic()||
|isPrivate()||
|isPublic()||
|isFinal()||
|isAnonymous()||
|isConstructor()||
|enclClass()|Returns owner|
|outermostClass()|Recurses upwards to get ClassSymbol of non-nested class|
|packge()|Returns enclosing PackageSymbol|
|isMemberOf(TypeSymbol, Types)|Can this class (first argument) see/use this member (MethodSymbol in our case) as one of its members?|
|isAccessibleIn(Symbol, Types)|First argument must be subclass of Symbol's owner! Checks if subclass can use this MethodSymbol.|
|getEnclosingElement()|returns owner|
|getKind()|returns ElementKind, simple enum value. Choose from CONSTRUCTOR, METHOD, STATIC_INIT_INSTANCE_INIT|
|getModifiers()|returns a `Set<Modifier>`. Modifier is an enum in javax.lang.model.element|

The most difficult methods are those who check the Symbol at hand against inheritance trees. These methods (there are a few more but `isMemberOf` and `isAccessibleIn` are rather typical) will help with resolving overrides.

## MethodSymbol - fields and methods

The previous fields and methods were generic methods from Symbol. MethodSymbol has a few more specific ones.

### Fields

The only relevant field is `params`, which is of type `List<VarSymbol>`. The field `Code code` doesn not contain source code, it has technical content related to the transition to bytecode.

### Methods

I make a distinction between methods related to inheritance and polymorphism and methods that just tell about the method as found in the code. I start with the latter category:

|Method|return type|Use|
|---|---|---|
|getModifiers()|`Set<Modifier>`|All modifiers in the form of enum values|
|isLambdaMethod()|boolean|True if flag LAMBDA_METHOD is 1|
|getParameters()|`List<VarSymbol>`|Calls 'params()'|
|params()|`List<VarSymbol>`|Returns list of VarSymbols|
|getKind()|ElementKind|Returns METHOD, STATIC_INIT, INSTANCE_INIT or CONSTRUCTOR|
|isStaticOrInstanceInit()|boolean|True if ElementKind is STATIC_INIT or INSTANCE_INIT|
|isVarArgs()|boolean|Returns true if flag VARARGS is 1|
|isDefault()|boolean|returns true if method is default method in interface|
|getReturnType()|Type|Returns return type|
|getThrownTypes()|`List<Type>`|Returns the thrown types from method signature|

These are methods related to inheritance, overriding etc. I omitted the last column ('Use') because I don't know what these methods exactly do. Note that `Types` is among the parameters, `Types` is a 5000 line utility class for operations on Types.

|Method|return type|
|---|---|
|implemented(TypeSymbol c, Types types)|Symbol|
|implementedIn(TypeSymbol c, Types types)|Symbol|
|binaryOverrides(Symbol _other, TypeSymbol origin, Types types)|boolean|
|binaryImplementation(ClassSymbol origin, Types types)|MethodSymbol|
|overrides(Symbol _other, TypeSymbol origin, Types types, boolean checkResult)|boolean|
|overrides(Symbol _other, TypeSymbol origin, Types types, boolean checkResult, boolean requireConcreteIfInherited)|boolean|
|isOverridableIn(TypeSymbol origin)|boolean|
|isInheritedIn(Symbol clazz, Types types)|boolean|
|implementation(TypeSymbol origin, Types types, boolean checkResult)|MethodSymbol|
|implementation(TypeSymbol origin, Types types, boolean checkResult, Predicate<Symbol> implFilter)|MethodSymbol|

## Methods that I need

To be able to analyze code, need a few collections where I can store method relations. Furthermore I need some methods that tell me more about the character of the method. Unlike javac's methods, I will heavily involve the contents of the method body.

### Call graph, forward and reverse

I want two maps of type `Map<MethodSymbol, Set<MethodSymbol>>`. One is called callgraphForward, the other one callgraphReverse. The former tells which methods are called by the method under study, the latter tells by which method the method under study is called. The latter map is probably pretty useful. I will store all method calls in it, both internal and external. As I cannot know which override is chosen (that is decided during runtime) I will just include the compile time type, even if this type only has the abstract form of the method.

In another map 'subtypes' of type `Map<Class, Set<Class>>` I store each class's subtypes, if there are any. This provides a clue for the possible implementation types. To exclude types that exist but are not instantiated anywhere, I create a list of non-instantiated types that are to be excluded from the possibilities.

### Methods I need (and why)

I will do one pass through all CompilationUnitTrees, especially the method bodies, to create a lot of `Set<MethodSymbol>` that help me to do a lookup. I want, for example, to be able to see which methods are involved in the linking of concrete implementations to abstract superclasses.

#### hasIterationWithoutMethodCalls

A method is included in the set if some sort of loop is to be found. It can be some for loop, something with iterate, a while loop or a do-while loop. Also included are methods where the functional programming stream() api is at work.

Reason for this is that I want to see where repetitive work is being done, especially with regards to object creation.

#### hasIterationWithMethodCalls()

Same as previous, but here connected to method calls to the outer world. If work in a loop is being delegated, I want to be able to quickly see what is being done as part of the loop.

#### hasConditional()

A conditional might indicate something important is being chosen. I want to be able to quickly filter all methods containing conditionals (yes, there will be a lot).

#### providesThisArgument()

A method that calls other methods using the `this` keyword is an indication that the two methods are highly interconnected.

#### propagatesArgument()

If you use the Strategy pattern, there is a good chance that you use an argument from the method as argument for the method call on the delegate. Generally, propagated arguments might tell you something about the use of the dependency.

#### makesAbstract()

A method that uses `new` to create an implementation can convert the runtime type to an more abstract supertype, for example by returning the new object while having its supertype as return type. Conversion can also take place with an assignment. Anyhow, every method where some form of active polymorphism takes place must be added to a dedicated set.

#### isRecursive()

Like looping, recursion might result in large numbers of newly created instances of some type. As I want to know which types are instantiated multiple times, knowing where recursion takes place is important.

Note that recursion can be a multi-method affair, with a cyclic call graph (which is allowed). There is no method in MethodSymbol that detects cyclic call graphs, so I might have to create some algorithm for it myself.

#### affectsState()

OOP is hard because of "state". Methods have side effects which makes the undertsanding of code a complex affair. Knowing is a method affects state might therefore be useful, especially if you can immediately see what part of "state" is affected.

Another benefit of this method is that it might help to identify utility classes, who tend to have methods that do not affect state.

#### isFunctional()

Same as previous but swaps true and false.

#### interactsWithFileSystem()

If an application interacts with the file system I want to know where this happens.

#### doesIoOperations()

Any other interaction with the outside world must be notified as well.

#### callsInnerMethod()

Maybe I want to be able to see how the inner-class callgraph works.

#### createsNewObject()

Important. All birth sites of new instances must be known. From every class I want to know where it is created, this tells a lot. It helps to identify the composition root, and tells you something about the nature of classes. If a class creates instances of classes with high centrality, the class might well be a composition root for example.

#### mainMethod()

I need to know where all main classes are right from the start. If there is a pom.xml and/or a MANIFEST.MF file, I want to know to what main method they point.





