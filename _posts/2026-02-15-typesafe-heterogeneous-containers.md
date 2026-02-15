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

Context, or `com.sun.tools.javac.util.Context`, contains a typesafe heterogeneous container which is initialized like this:

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

Below is the implementation. It is the type parameter T that ensures that the type of key matches the type of data.

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

### What you can do with Key as key

As Key is a reference type, nothing will hold you back from using multiple instances of Key with the same type parameter in the same map. Keys must be unique, but if each Key is a different reference object, it is not a problem if they are identical in their type parameter. In the follwing snippet I have illustrated that it is perfectly possible to create very identical keys that nevertheless can co-exist as keys in a hashmap.

```
class Key<T>{}

Key<String> k1 = new Key<>();
Key<String> k2 = new Key<>();

System.out.println(k1==k2);  // false. Although same type param, identity differs

Map<Key<?>, Object> myMap = new HashMap<>();

myMap.put(k1, "Hello");
myMap.put(k2, "Bye");

System.out.println(myMap); // {REPL.$JShell$2$Key@3b94d659=Hello, REPL.$JShell$2$Key@1f021e6c=Bye}
```

In Context, no Key objects with the same type parameter will coexist in the hashmap because the Key field in the various classes is static and thus the Key is the same for all instances. But if you would make the Key field non-static, you could perfectly use similar Key objects as keys in the hashmap.

## A comparison

My first thought on these two ways of creating and using typesafe heterogeneous containers was that the javac variant (Context) was superior because it doesn't rely on reflection. But ChatGPT has another take namely this:

- Context is internal and knows for sure that `Key<T>` has a value of type T
- Bloch's container is meant for public use and cannot rely on proper setting of key-value pairs
- The cast method is from the Java Reflection API but doesn't take much overhead

### Reifiable and non-reifiable

Furthermore, context can differentiate between `List<String>` and `List<Integer>`, while Bloch's container can't. This is because `Class<T>` is fundamentally different from `Key<T>`. `List<Integer>` is a so-called non-reifiable type, meaning that its compile type differs from its runtime type (type erasure cause List<Integer> to become List). Context's container supports non-reifiable types, Bloch's container doesn't. 

This means that if you want to be able to store the same type in the container, only with a different generic parameter, you can only do so with th eContext container. Furthermore, if you want multiple keys with the same type, you can also do that with the Context container, but then you need to make the Key field non-static. As it is now, with a static field for Key, each class will only have one key, no matter how many instance of that class are generated.

## Interesting javadoc comment

Context contains this 'get' class:

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

Notice the 'uncheckedCast' method at the end. The comment says it cannot fail, which is true because upon insertion in the map, a compile-time check was done that ensured that the type parameter of the key matched the type of the value. Below is the 'uncheckedCast' method, and look what its javadoc comment says:

```
    /**
     * TODO: This method should be removed and Context should be made type safe.
     * This can be accomplished by using class literals as type tokens.
     */
    @SuppressWarnings("unchecked")
    private static <T> T uncheckedCast(Object o) {
        return (T)o;
    }
```

ChatGPT told me that he thinks that the makers are aware of the lack of runtime safety and therefore, to make the code even more fail-safe, want to move to a method that is more robust, more like Bloch's implementation. The cast `(T)o` can throw an exception but this exception is suppressed by the annotation. Even though the Java compiler will never fail because of this, there is still some notion that risks should be minimized even more.







