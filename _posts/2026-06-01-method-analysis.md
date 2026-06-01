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
|33|PARAMETER|Flag for VarSymbol that is method parameter|
|34|VARARGS|Flag that marks method having varargs parameter|
|43|DEFAULT|Marks a default method or an interface containing default methods|
|51|MODULE|Flag to indicate class symbol is for module-info|
|52|AUTOMATIC_MODULE|Flag to indicate the given ModuleSymbol is an automatic module|
|17|BODY_ONLY_FINALIZE|Flag that marks finalize block as body-only|
|61|RECORD|Class is a record|

Note: to many of these flags there is a method that returns whether the flag, or multiple flags, are set. If there is an appropriate method available, you do not have to access the flags_field field yourself.

Note: there is an enum ElementKind in the accessible api (javax.lang.model.element.ElementKind). This enum value can be requested using all sorts of `getKind()` methods. ElementKind does not go as deep into compiler processes as Flags but tells you correctly what sort of Symbol you are dealing with, including the differences between different ClassSymbols (CLASS, RECORD, INTERFACE, ENUM) or between different VarSymbols (FIELD, LOCAL_VARIABLE, PARAMETER etc).

#### Name name

#### Type type

#### Symbol owner

#### Completer completer

#### Type erasure_field 