# Semantic resolution

I came across various javac classes that are being used when javac tries to resolve symbols and types. This is very much an implementation thing, but it would be unfortunate if I wouldn't write down some things that I just learned.

## Env, Scope, Scope, JavacScope, AttrContext

There are multiple classes, some abstract, and an interface, that require discussion here:

- com.sun.tools.javac.code.Scope
- com.sun.tools.javac.comp.Env
- com.sun.tools.javac.comp.AttrContext
- com.sun.source.tree.Scope
- com.sun.tools.javac.api.JavacScope

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

If we discard com.sun.source.tree.Scope and com.sun.tools.javac.api.JavacScope, which serve as a sort of api for external use, it is remarkable that there are multiple classes involved in the same process of symbol and type resolution. Instead of wrapping Scope in AttrContext and AttrContext in Env, you could simply put everything in Env.

Generally, it doesn't matter a lot, as you would end up with the same amount of code lines and problems to solve, so it is not a big deal. But what we can do is try to find out what, generally speaking, the strength of each of this classes is.

## Scope

I start with the one that is wrapped the most. It is also the largest class in terms of lines, namely 1145. Env stops at 154 and AttrContext at 181. Scope is a sophisticated class designed for performance. This is what the javadoc comment on top says:

_A scope represents an area of visibility in a Java program. The Scope class is a container for symbols which provides efficient access to symbols given their names. Scopes are implemented as hash tables with "open addressing" and "double hashing". Scopes can be nested. Nested scopes can share their hash tables._

### Structure

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

### Fields and methods

I discuss two Scope classes, namely the main class (Scope itself) and static inner class ScopeImpl. The latter one might be the most relevant subclass.

#### Top level (Scope)

The Scope top class has three field of which 'owner' is the most important:

```
public final Symbol owner;

// two other fields
private static final Predicate<Symbol> noFilter = null;
ScopeListenerList listeners = new ScopeListenerList();
```

The method list is all about retrieving symbols, eventually based on some specific condition, or checking if certain symbols exist somewhere in the current or the outer scope, or checking whether the current scope contains any symbols at all. I only wrote down the signatures for brevity.

```
    public final Iterable<Symbol> getSymbols();
    public final Iterable<Symbol> getSymbols(Predicate<Symbol> sf);
    public final Iterable<Symbol> getSymbols(LookupKind lookupKind);
    public abstract Iterable<Symbol> getSymbols(Predicate<Symbol> sf, LookupKind lookupKind);
    public final Iterable<Symbol> getSymbolsByName(Name name);
    public final Iterable<Symbol> getSymbolsByName(final Name name, final Predicate<Symbol> sf);
    public final Iterable<Symbol> getSymbolsByName(Name name, LookupKind lookupKind);
    public abstract Iterable<Symbol> getSymbolsByName(final Name name, final Predicate<Symbol> sf, final LookupKind lookupKind);
    public final Symbol findFirst(Name name);
    public Symbol findFirst(Name name, Predicate<Symbol> sf);
    public boolean anyMatch(Predicate<Symbol> filter);
    public boolean includes(final Symbol sym);
    public boolean includes(final Symbol sym, LookupKind lookupKind);
    public boolean isEmpty();
    public abstract Scope getOrigin(Symbol byName);
    public abstract boolean isStaticallyImported(Symbol byName);
```

#### ScopeImpl

ScopeImpl has all sorts of methods, many of them implementations of the abstract methods of either Scope or WritableScope. The methods and fields that are not inherited and thus unique for this static inner class are all related to a so-called hash table construct that is used for efficient traversal of shadowed symbols.

The hash table is stored as a field:

```
Entry[] table;
```

Entry is a private static inner class with this code:

```
    private static class Entry {

        //The referenced symbol. sym == null   if   this == sentinel
        public Symbol sym;

        //An entry with the same hash code, or sentinel.
        private Entry shadowed;

        // Next entry in same scope.
        public Entry nextSibling;

        // Prev entry in same scope.
        public Entry prevSibling;

        // The entry's scope.scope == null   iff   this == sentinel
        public ScopeImpl scope;

        public Entry(Symbol sym, Entry shadowed, Entry nextSibling, ScopeImpl scope) {
            this.sym = sym;
            this.shadowed = shadowed;
            this.nextSibling = nextSibling;
            this.scope = scope;
            if (nextSibling != null)
                nextSibling.prevSibling = this;
        }

        // Return next entry with the same name as this entry, proceeding outwards if not found in this scope.
        public Entry next() {
            return shadowed;
        }

        public Entry next(Predicate<Symbol> sf) {
            if (shadowed.sym == null || sf == null || sf.test(shadowed.sym)) return shadowed;
            else return shadowed.next(sf);
        }

    }
```

This means that there exists an array of Entry objects, and these objects have fields that point to other Entry objects in the same array. There are three methods responsible for two types of link chain: `private Entry shadowed;` for when you need to find symbols that shadow other symbols, and `public Entry nextSibling;` and `public Entry prevSibling;` to quickly search through the contents of the array.

The fields related to this mechanism, other than the previously mentioned 'table', are the following:

```
// A linear list that also contains all entries in reverse order of appearance (i.e later entries are pushed on top).
public Entry elems;

// Mask for hash codes, always equal to (table.length - 1).
int hashMask;

// The hash table's initial size.
private static final int INITIAL_SIZE = 0x10;

// Use as a "not-found" result for lookup. Also used to mark deleted entries in the table.
private static final Entry sentinel = new Entry(null, null, null, null);
```

The methods related to the hash table are:

```
private void dble()  // doubles the size of the hash table
int getIndex (Name name)  // finds the index of a specific Symbol if its name is provided
protected Entry lookup(Name name)  // finds the entry belong to a specific Symbol, if you provide its name
```

## AttrContext

Env is a class that wraps AttrContext and Scope. AttrContext contains a lot of fields, of which Scope is one. So all the data of Scope is in AttrContext. Apart from that AttrContext has the following extra fields:

```
    /** The scope of local symbols.
    WriteableScope scope = null;

    /** The number of enclosing `static' modifiers.
    int staticLevel = 0;

    /** Is this an environment for a this(...) or super(...) call?
    boolean isSelfCall = false;

    /** are we analyzing the arguments for a constructor invocation?
    boolean constructorArgs = false;

    /** Are we evaluating the selector of a `super' or type name?
    boolean selectSuper = false;

    /** Is the current target of lambda expression or method reference serializable or is this a serializable class?
    boolean isSerializable = false;

    /** Is this a serializable lambda?
    boolean isSerializableLambda = false;

    /** Is this a lambda environment?
    boolean isLambda = false;

    /** Is this a speculative attribution environment?
    AttributionMode attributionMode = AttributionMode.FULL;

    /** Is this an attribution environment for an anonymous class instantiated using <> ?
    boolean isAnonymousDiamond = false;

    /** Is this an attribution environment for an instance creation expression?
    boolean isNewClass = false;

    /** Indicate if the type being visited is a service implementation
    boolean visitingServiceImplementation = false;

    /** Indicate protected access should be unconditionally allowed.
    boolean allowProtectedAccess = false;

    /** Are arguments to current function applications boxed into an array for varargs?
    Resolve.MethodResolutionPhase pendingResolutionPhase = null;

    /** A record of the lint/SuppressWarnings currently in effect
    Lint lint;

    /** The variable whose initializer is being attributed useful for detecting self-references in variable initializers
    Symbol enclVar = null;

    /** ResultInfo to be used for attributing 'return' statement expressions (set by Attr.visitMethod and Attr.visitLambda)
    Attr.ResultInfo returnResult = null;

    /** ResultInfo to be used for attributing 'yield' statement expressions (set by Attr.visitSwitchExpression)
    Attr.ResultInfo yieldResult = null;

    /** Symbol corresponding to the site of a qualified default super call
    Type defaultSuperCallSite = null;

    /** Tree that when non null, is to be preferentially used in diagnostics. Usually Env<AttrContext>.tree is the tree to be referred to in messages, but this may not be true during the window a method is looked up in enclosing contexts (JDK-8145466)
    JCTree preferredTreeForDiagnostics;
```

There are just a few methods, no setters and getters are required as all fields are package-private.

## Env

An important fact is that while Scope is related to a namespace (block, method body, class body etc), Env is related to a specific AST Tree ('node'). Env provides all the relevant semantic context for that specific Tree, while Scope gives access to all the available symbols in the namespace in which that Tree lives.

You can think of a situation where, somewhere in a method body, a specific variable (Symbol) is declared halfway, while the code analysis has not reached that point yet. In the logic of Env, it is relevant where in the body you are, while for Scope, it doesn't matter. If the code analysis progresses, new Env objects are being created (using a lot of smart reuse of already gathered data). But while the analysis progresses, the Scope object will remain the same, as long as you do not exit the namespace.

This is what ChatGPT told me:

_Scopes can be reused across multiple Env objects because many AST nodes live in the same lexical scope, but each node may still need its own Env (its own semantic context)._

There are all sorts of 'dup' methods that smartly copy elements/fields of Env when a new Env needs to be created that can reuse a lot of the material of the previous Env object.

### Fields

These are the fields of Env:

```
    /** The next enclosing environment.
    public Env<A> next;

    /** The environment enclosing the current class.
    public Env<A> outer;

    /** The tree with which this environment is associated.
    public JCTree tree;

    /** The enclosing toplevel tree.
    public JCTree.JCCompilationUnit toplevel;

    /** The next enclosing class definition.
    public JCTree.JCClassDecl enclClass;

    /** The next enclosing method definition.
    public JCTree.JCMethodDecl enclMethod;

    /** A generic field for further information.
    public A info;

    /** Is this an environment for evaluating a base clause?
    public boolean baseClause = false;
```

As you see the specific relevant Tree objects are included as fields, and there are references to the Env objects belonging to the current class and to the next enclosing environment.

### Methods

If you look at the methods, I found this an interesting one:

```
    /** Return closest enclosing environment which points to a tree with given tag.
    public Env<A> enclosing(JCTree.Tag tag) {
        Env<A> env1 = this;
        while (env1 != null && !env1.tree.hasTag(tag)) env1 = env1.next;
        return env1;
    }
```

You can use this method to go up in the hierarchy until you find an Env object related to a Tree that has a specific tag (like CLASSDEF or METHODDEF or WHILELOOP). 

Furthermore there are the 'dup' and 'dupTo' methods. It is sort of one method with overload etc:

```
    /** Duplicate this environment, updating with given tree and info,
     *  and copying all other fields.
    public Env<A> dup(JCTree tree, A info) {
        return dupto(new Env<>(tree, info));
    }

    /** Duplicate this environment into a given Environment,
     *  using its tree and info, and copying all other fields.
    public Env<A> dupto(Env<A> that) {
        that.next = this;
        that.outer = this.outer;
        that.toplevel = this.toplevel;
        that.enclClass = this.enclClass;
        that.enclMethod = this.enclMethod;
        return that;
    }

    /** Duplicate this environment, updating with given tree,
     *  and copying all other fields.
    public Env<A> dup(JCTree tree) {
        return dup(tree, this.info);
    }
```

## Summary

Javac has a sophisticated of resolving Symbols. While parsing code into an AST can be done mostly line by line without further context, symbol resolution requires exploration of this context. Env, AttrContext and Scope are central classes in this process. 