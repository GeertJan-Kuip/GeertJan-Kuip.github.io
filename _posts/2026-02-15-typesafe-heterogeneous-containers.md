# Typesafe heterogeneous containers

In the previous blog post I encountered a so called heterogeneous typesafe container in the Context type that is being used in javac. This term comes form Joshua Bloch's book 'Effective Java' and is found under item 33 in the third edition of 2018.

In this post I will discuss Bloch's example, then I will compare it to what I found in Context and I will get back to the source code of Maven, because as I remember there is a variant of it in there as well.

## Effective Java

In item 33, part of the chapter on generics, Bloch encourages the reader to make use of typesafe heterogeneous containers. Bloch distinguishes these typesafe heterogeneous containers from parameterized containers such as `List<T>` and `Map<String, Double>`. Those are typesafe but not heterogeneous. Heterogeneous means that you can put objects of different types in the same container. 

Of course you can put heterogeneous objects in a parameterized container, for example like `List<Object>` or `Map<String, Object>`, but those are not typesafe. Type safety means that when you put a wrong type in a container the warning will occur during compilation, not during runtime.

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


