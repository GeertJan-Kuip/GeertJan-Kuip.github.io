# Method analysis

With Symbol, the forbidden-access class in jdk.compiler found as com.sun.tools.javac.code.Symbol, you can get some information about methods by using the methods and fields belonging to either subclass MethodSymbol or to Symbol itself.

In this blog post I am going to provide a list of information about MethodSymbols that is directly available, whereby the `flags_field` field of Symbol is important. After that I will provide a set of other pieces of information about methods that I want to generate.

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

These are methods related to inheritance, overriding etc. I omitted the last column ('Use') because I don't know what these methods exactly do:

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





