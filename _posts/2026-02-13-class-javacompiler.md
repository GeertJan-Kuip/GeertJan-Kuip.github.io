# Class JavaCompiler

In the jdk.compiler module there is package com.sun.tools.javac.main which contains JavaCompiler. In this blog post I will discuss its contents, encouraged by ChatGPT who told me that this class gives good insight in the difffernt phases of compilation.

Having said that, I know that I will encounter a lot of mechanisms within classes directly related to JavaCompiler. I am going to dive in these rabbit holes without reservations, hoping to learn a lot more about advanced Java programming.

By writing this post, I hope to learn more about the folwoing topics:

- The role, use and internals of related classes like Context, Enter, Symtab, Classreader
- The specifics of different phases
- How JavaCompiler relates to JavadocTool, interface JavaCompiler and JavacTaskImpl
- The reason why multiple instances of JavaCompiler are being created during compilation
- How comments and annotations are part of this

Btw JavaCompiler has all sorts of layers on top of it, you do not access it directly. And this class should not be confused with public interface JavaCompiler (javax.tools.JavaCompiler) which has com.sun.tools.javac.api.JavacTool as runtype (thus not this JavaCompiler class).

## Overall structure

JavaCompiler looks like it is well structured. Every required dependency has a proper field with proper javadoc comments, making it easier to understand what the class is doing and how it relies on other classes.

The general structure, omitting a lot, can be summarized as this:

```
field Context.Key<JavaCompiler> compilerKey

normal constructor + .instance method to retrieve JavaCompiler belonging to specific context

many fields (Symtab, Source, Flow, Lower, Gen, ModuleFinder etc)

many boolean flags (verbose, sourceOutput etc) 

method compile(...)  // this is the main method

lots of other methods and stuff
```

### Context

The first line of JavaCompiler is this:

```
public static final Context.Key<JavaCompiler> compilerKey = new Context.Key<>();
```

The first thing to know is that this line can be found in (almost) identical form in all other classes that play lead roles during compilation. Think of Symtab, ClassFinder and Enter.

The essential thing here is that Context has a field `protected final Map<Key<?>,Object> ht = new HashMap<>();`. In this map one and only one instance of all required classes is stored. To guarantee uniqueness and to be able to store instances of different types, Context uses a trick. It has a static inner class named Key with a type parameter:

```
    public static class Key<T> {
        // note: we inherit identity equality from Object.
    }
```

Every instance being used for the compilation process generates a Key object upon creation, like in the first line of JavaCompiler. This Key object is unique by definition, as it is a refernce object (and not a String).

The constructor of each of these classes is protected and will not be called directly. To instantiate a class, an instance method must be called. This is the instance method for JavaCompiler, all other classes participating in this construction have the same instance method:

```
    public static JavaCompiler instance(Context context) {
        JavaCompiler instance = context.get(compilerKey);
        if (instance == null)
            instance = new JavaCompiler(context);
        return instance;
    }
```

This method checks if the instance is already added to Context context. If not, the constructor is being called. This constructor has this line:

```
    context.put(compilerKey, this);
```

So basically it adds itself to the map in Context where all the instances of the various classes live. The `put` method in COntext is quite sophisticated:

```
    public <T> void put(Key<T> key, T data) {
        if (data instanceof Factory<?>)
            throw new AssertionError("T extends Context.Factory");
        checkState(ht);
        Object old = ht.put(key, data);
        if (old != null && !(old instanceof Factory<?>) && old != data && data != null)
            throw new AssertionError("duplicate context value");
    }
```

What is so great about this is the generics. Remember that the map in which instances are stored is ot type `Map<Key<?>,Object>`. The Key instance has a generic parameter that must match the type of the Object. This type similarity is guaranteed because the `put` method enforces type equality. Key must have T as type, and Object must be of type T.

This compile type safety is just really clever. In the end we have a context, stored as a Map of key value pairs, with one guaranteed unique instance of every object type that is required for the compilation process.

And if we need that object, we just use Context's get method:

```
    public <T> T get(Key<T> key) {
        checkState(ht);
        Object o = ht.get(key);
        if (o instanceof Factory<?> fac) {
            o = fac.make(this);
            if (o instanceof Factory<?>)
                throw new AssertionError("T extends Context.Factory");
            Assert.check(ht.get(key) == o);
        }

        /* The following cast can't fail unless there was
         * cheating elsewhere, because of the invariant on ht.
         * Since we found a key of type Key<T>, the value must
         * be of type T.
         */
        return Context.uncheckedCast(o);
    }
```

There are some nested methods in it but in the end it is guaranteed to work.



    