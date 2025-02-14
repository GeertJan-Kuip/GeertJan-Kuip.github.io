## Get the primitive out of the box

I tried to define some rules of thumb about primitive/reference conversion that help me to find mistakes in 1Z0-819 questions. Everything about type conversion can be found in [chapter 5 of the Java Language Specification](https://docs.oracle.com/javase/specs/jls/se23/html/jls-5.html) but that is such a long set of rules that I might, for new, be better of with something more simple and generic.

### Rule 1
**Boxed primitives/reference types don't get along very well. They behave hierarchically.**

### Rule 2
**Primitives get along very well. They convert smoothly, in case of potential information loss you need to do casting.

Okay, this is quite generalistic but it might summarize things well. Now a bit more detail:

- Primitive conversion, both widening and narrowing, almost always works well. If it doesn't compile, the reason is that the information loss is too large and the compiler thinks you are unaware of this. In this case you use type casting and voila.

- Reference conversion (from one boxed type to the other) almost never works. Classes are aware of hierarchical relationships so the only conversion that works is from subclasses of Number to Number or vice versa. The latter only works if the primitive stored in the number object is the same type as the primitive type that is stored in the subclass. In all cases explicit casting is required.

- Reference types refuses to store a primitive that is not their type. Primitive types have no problem doing a conversion during assignement, as long as it doesn't result in information loss. For example:

```
float a = 10L; //this works

Float a = 10L; //this does not
```

- Conversions from boxed/reference type to primitive type can be done without explcit casting if the conversion is not a narrowing conversion:

```
Float a = 10.0f;
double b =  a;
```

- If a narrowing casting has to be done from reference type to primitive type, the conversion has to start with a cast to the primitive equivalent of the reference type. Both examples show a cast from one reference type to another:

```
Integer a = 10;
Short b = (short)(int) a; // fine
```
```
Integer a = 10;
Short b = (short) intValue(a); // also fine
```

Okay, these were the rules of thumb. Primitives behave better than boxed types.

