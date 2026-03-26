# Semantic resolution

I came across various javac classes that are being used when javac tries to resolve symbols and types. This is very much an implementation thing, but it would be unfortunate if I wouldn't write down some things that I just learned.

## Env, Scope, Scope, JavacScope, AttrContext

There are multiple classes, some abstract, and an interface, that require discussion here:

- com.sun.tools.javac.code.Scope
- com.sun.tools.javac.comp.Env
- com.sun.tools.javac.comp.AttrContext
- com.sun.source.tree.Scope (interface)
- com.sun.tools.javac.api.JavacScope

I was confused about the name Scope being reused, ChatGPT explained me how and why this happened. For the compiler to work, it uses `Scope` objects (the first in the list above) and `Env<AttrContext>` objects. The hierarchy in them is as follows:

Env -> AttrContext -> Scope

In words: Env depends on AttrContext, AttrContext depends on Scope. Or: AttrContext wraps Scope, and Env wraps AttrContext.

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

 




