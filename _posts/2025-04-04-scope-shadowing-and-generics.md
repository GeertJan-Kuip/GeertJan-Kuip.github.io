## Scope, shadowing and generics

Short post. I was wondering whether the declaration of a generic type in a class declaration leaves room to create methods that have their own generic type declaration and if so, if that generic type declaration of the method shadows the generic type declaration of the class. 

Short answer: it does and it does.

The following code shows an example where the \<T\> of the method shadows the \<T\> of the class. 

```
public class Generic<T> {

    public <T> T justPassOn(T t){
        return t;
    }

    public static void main(String[] args){
        Generic<String> gen = new Generic<>();
        Integer i = gen.<Integer>justPassOn(Integer.valueOf(5));  // The <Integer> can be omitted, compiler will infer
        System.out.println(i);
    }
}
```

As you see, the generic method can use another type than that of the instance it is operating on. There are two generic types involved, both under the name of T.

If we would have wanted the method 'justPassOn' for each instance to have the same type as the object, we would have to write a method declaration like this, without type parameter section \<T\>:

```
public T justPassOn(T t){
```

If we wanted to have better readable code, we should not get into the realm of shadowing and use another letter as type parameter:

```
public <U> U justPassOn(U u){
```

But this is how it fundamentally works. A method is independent of its class with regards to generics, it can have its own declaration of generic type. This is btw not true for fields. You cannot do this:

```
public class Generic<T> {

    <U> U u;  // not good, uncompileable
}
```

By the way it also works like this in interfaces, and it is probably terrible practice:

```
interface myInterface<T>{  // this <T> has no role since it is shadowed by the <T> of the method.  
    <T> T work();
}
```

The end.




