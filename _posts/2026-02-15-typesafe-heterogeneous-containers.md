# Typesafe heterogeneous containers

In the previous blog post I encountered a so called heterogeneous typesafe container in the Context type that is being used in javac. This term comes form Joshua Bloch's book 'Effective Java' and is found under item 33 in the third edition of 2018.

In this post I will discuss Bloch's example, then I will compare it to what I found in Context.

## Effective Java

In item 33, part of the chapter on generics, Bloch encourages the reader to make use of typesafe heterogeneous containers. Bloch distinguishes these typesafe heterogeneous containers from parameterized containers such as `List<T>` and `Map<String, Double>`. Those are typesafe but not heterogeneous. Heterogeneous means that you can put objects of different types in the same container. 

Of course you can put heterogeneous objects in a parameterized container, for example like `List<Object>` or `Map<String, Object>`, but those are not typesafe. Type safety means that when you put a wrong type in a container the warning will occur during compilation, not during runtime.

### What Bloch proposes

Bloch's typesafe heterogeneous container is a map, whereby the value is of type `Object` and the key is of type `Class<T>`, whereby for every entry in the map the generic T of the key must match the type of the value. Storing an object that is not of the type indicated by the key must result in a compile time warning. This is how Bloch implements this:

```
// Typesafe heterogeneous container pattern - implementation
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

The put method guarantees that the key and value are of the same generic type. That type can be anything. The [Objects.requireNonNull(type)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Objects.html#requireNonNull(T)) is added to prevent the key to be null (it throws NullPointerException if the key is null). 

The getFavorite method also ensures that key and value are of the same type, or at least of compatible type. If not, the [cast](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html#cast(java.lang.Object)) method wil throw a ClassCastException. The cast method, like all methods from `Class<t>` is from the Java Reflection API. You can use it in your code if no non-refelction alternatives are available.

### About class literals and reflection

I am not used to working with `Class<T>` but I just learned that you can do this:

```
Class<Integer> c = Integer.class;  // Integer.class is called a class literal
System.out.println(c); // prints "class java.lang.String"

System.out.println(Integer.class instanceof Class<Integer>); // prints true
```

`Class<T>` has a ton of methods that let you inquire all sorts of properties of a specific class (getMethods(), getFields() etc). These methods rely on reflection and I'm not going to dig into this, but it might be an interseting topic for the future. Reflection, as stated earlier, is a sort of last resort, because performance, safety and maintainability are not great.

### The problem with Bloch's proposal

As keys must be unique, the typesafe heterogeneous container Bloch proposes can only have one entry per type. It is not possible to store multiple entries of type Integer for example, as it will lead to duplicate keys. See this code example:

```
1 Class<Integer> c = Integer.class;
2 Map<Class<?>, Object> mm = new HashMap<>();
3 mm.put(Integer.class, 7);
4 mm.put(Integer.class, 8);
5 mm.put(String.class, "Hello");
6 System.out.println(mm);
7
output: {class java.lang.Integer=8, class java.lang.String=Hello}
```

Line 5 is overwritten by line 4, only one Integer.type is in the resulting map.

## Context

Context, or `com.sun.tools.javac.util.Context`, contains a typesafe heterogeneous container with is initialized like this:

```
    protected final Map<Key<?>, Object> ht = new HashMap<>();
```

Instead of `Class<?>` it has `Key<?> as key. This `Key<?>` is a static inner class in Context:

```
    public static class Key<T> {
        // note: we inherit identity equality from Object.
    }
```

The body is empty and  you can create a Key object like this:

```
    Context.Key<SomeClass> someClassKey = new Context.Key<>();
```

Every class that needs itself to be in the Context ht map has a protected static final field similar to this:

```
    protected static final Context.Key<Symtab> symtabKey = new Context.Key<>();
```

To put a Key with a class instance in the ht map, this put method is being used:

```
context.put(symtabKey, this);
```

As the key is a static final variable, it is only created once and never modified. So every class has just one Context.Key with its own type as generic type parameter. The way that such a class generates an instance of itself is via two methods. Instance creation is done with the instance method, which looks into the Context ht container to see if its key (of which there is only one per type) is in it. If not, it calls the constructor and puts itself in it:

```
    public static Symtab instance(Context context) {
        Symtab instance = context.get(symtabKey);
        if (instance == null)
            instance = new Symtab(context);
        return instance;
    }

    protected Symtab(Context context) throws CompletionFailure {
        context.put(symtabKey, this);
        // more code relevant for specific class
    }
```

## A comparison

My first thought on these two ways of creating and using typesafe heterogeneous containers was that the javac variant (Context) was superior because it doesn't rely on reflection. But ChatGPT has another take namely this:

- Context is internal and knows for sure that `Key<T>` has a value of type T
- Bloch's container is meant for public use and cannot rely on proper setting of key-value pairs

The method 





