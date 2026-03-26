# Semantic resolution

I came across various javac classes that are being used when javac tries to resolve symbols and types. This is very much an implementation thing, but it would be unfortunate if I wouldn't write down some things that I just learned.

## Env, Scope, Scope, JavacScope, AttrContext

There are multiple classes, some abstract, and an interface, that require discussion here:

- com.sun.tools.javac.code.Scope
- com.sun.tools.javac.comp.Env
- com.sun.tools.javac.comp.AttrContext
 

I was confused about the name Scope being reused, ChatGPT explained me how and why this happened. For the compiler to work, it uses `Scope` objects (the first in the list above) and `Env<AttrContext>` objects. The hierarchy in them is as follows:

Env -> AttrContext -> Scope

In words: Env depends on AttrContext, AttrContext depends on Scope. Or: Env wraps AttrContext and AttrContext wraps Scope.

### Why two Scope types

Then the question is, why is there an extra interface for Scope, with an implementation in JavacScope? According to ChatGPT, this JavacScope class is a wrapper around `Env<AttrContext>` and is meant to have some accessible form of the Env and Scope info. In Trees you have this method:

```
public abstract Scope getScope(TreePath path);
```

It returns a com.sun.source.tree.Scope object, which is meant for public use (people like me). It is useful if you want to know which symbols are available in the given scope, and it can give you the Element/Symbol of the enclosing class or method.

```
public interface Scope {
    /**
     * Returns the enclosing scope.
     * @return the enclosing scope
     */
    public Scope getEnclosingScope();

    /**
     * Returns the innermost type element containing the position of this scope.
     * @return the innermost enclosing type element
     */
    public TypeElement getEnclosingClass();

    /**
     * Returns the innermost executable element containing the position of this scope.
     * @return the innermost enclosing method declaration
     */
    public ExecutableElement getEnclosingMethod();

    /**
     * Returns the elements directly contained in this scope.
     * @return the elements contained in this scope
     */
    public Iterable<? extends Element> getLocalElements();
}
```

## Why there are three objects for one job

If we discard com.sun.source.tree.Scope and com.sun.tools.javac.api.JavacScope, which serve as a sort of api for external use, it is remarkable why there are multiple classes involved in the same process of symbol and type resolution. Instead of wrapping Scope in AttrContext and AttrContext in Env, you could simply put everything in Env.

Generally, it doesn't matter a lot, as you would end up with the same amount of code lines and problems to solve, so it is not a big deal. But what we can do is try to find out what, generally speaking, the strength of each of this classes is.

### Scope

I start with the one that is wrapped the most. It is also the largest class in terms of lines, namely 1145. Env stops at 154 and AttrContext at 181. Scope is a sophisticated class designed for performance. This is what the javadoc comment on top says:

_A scope represents an area of visibility in a Java program. The Scope class is a container for symbols which provides efficient access to symbols given their names. Scopes are implemented as hash tables with "open addressing" and "double hashing". Scopes can be nested. Nested scopes can share their hash tables._

#### Structure

The Scope class has all sorts of nested static classes with a lot of inheritance relationships. No static inner class is nested more than one level deep, but inheritance relationships do run deeper. This is the inheritance tree:

```
Scope
  |
  |- CompoundScope
  |   |
  |   |- MemberScope
  |   |- ImportScope
  |       |
  |       |- StarImportScope
  |       |- NamedImportScope
  |  
  |- WritableScope
  |   |  
  |   |- ScopeImpl
  |       |
  |       |-ErrorScope
  |
  |- SingleEntryScope
  |
  |- FilterImportScope
```

For some reason, MemberScope is not a static inner class of Scope but of MembersClosureCache in Types, which is a utilty class in com.sun.tools.javac.code. 

What this tree suggests is that the nested structure of scopes has an order to it which is represented by the various static inner classes. 

Furthermore, Scope has some inner classes, interfaces and enums that are not children of Scope itself. They are:

```
Entry             - private static class
ImportFilter      - public interface 
LookupKind        - public enum 
ScopeListener     - public interface 
ScopeListenerList - public static class 
```

All together there are 15 subitems, of which one (MemberScope) lives somewhere else.

####  
