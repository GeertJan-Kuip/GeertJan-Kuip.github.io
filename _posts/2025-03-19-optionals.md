## Optionals

Optional is an object type that can either be empty or contains a non-null value. Introduced in versopn 8, it tried to soften the problems with NullPointerException by providing a new way to deal with the absence of a value. It is best to use it as a return value, whereby it makes code better readable, communicating to readers that some method might well return no value at all.

In the context of functional programming and streams, Optional<T> is the return value of .max(), .min(), .findFirst(), .findAny() and .reduce(). For the latter, only the reduce method with one parameter returns an Optional. Optional fits well into streams, it has a .stream() method that returns a stream and .map(), .flatMap() and .filter() that return a new Optional. 

### Inquiring Optionals

Apart from the stream context, any Optional type value can be inquired to check whether it is empty or not and if not, what value it contains. These are the methods: 

|Method|Description|
|----|----|
|**_get_**()|If Optional is empty throws exception, otherwise returns value|
|**_ifPresent_**(Consumer c)|Does nothing if empty, calls consumer with value if not empty|
|**_isPresent_**()|Returns true or false|
|**_orElse_**(T other)|Returns value or other parameter if empty|
|**_orElseGet_**(Supplier s)|If empty returns result of supplier, otherwise returns value|
|**_orElseThrow_**()|Throws NoSuchElementException if empty, otherwise returns value|
|**_orElseThrow_**(Supplier s)|Throws exception created by calling supplier, otherwise returns value|

Generally, 6 of the 7 methods return the value if the Optional is not empty. The only one that doesn't is ifPresent() which calls a Consumer with value. If the Optional is empty, they all do something different:

- Nothing (ifPresent(Consumer c))
- Throw exception (get(), orElseThrow(), orElseThrow(Supplier s))
- Return false (isPresent())
- Return result of a Supplier (orElseGet(Supplier s))
- Return the argument T of the method (orElse(T other))

In words, you have the option to return nothing, false, an alternative or an exception. The point of all this is that you can write code that deals with Optionals without have to write if-statements or ternary operations. It promotes readability.

Note: I realized that .get() and .orElseThrow() actually do exactly the same. Perplexity told me that it is better to use .orElseThrow() because it has a name that improves code readability. Optional.get() was part of the 2008 introduction, .orElseThrow() came in version 10.

### Creating Optionals

Other than Java methods returning Optionals, you can create them yourself as well. There are the three static methods available:

- Optional.**_of_**(T value)
- Optional.**_empty_**()
- Optional.**_ofNullable_**(T value)

Optional.of(T value) can not be set with a null value, it will throw an exception then. Optional.empty() returns an empty Optional. Optional.ofNullable(T value) can be set with either a value or null, upon which an empty Optional is created. 

### Optionals in streams

You can use .stream() on an Optional. It creates a stream consisting of the value, or an empty stream if the Optional is empty. Furthermore there are the methods .map(), .flatMap() and .filter() that can be used on an Optional and they return a new Optional. The first two are part of the exam.

Optional.map(Function\<? super T,? extends U\> mapper) returns an Optional\<U\> containing the return value of the Function. This can be problematic if the function itself returns an Optional, you get an Optional within an optional then. This won't compile.

To prevent this, use .flatMap(Function\<? super T,? extends Optional\<? extends U\>\> mapper) instead. FlatMap has Optional\<u\> as return type as well, but removes one wrapping layer so you don't end up with a non-compilable nested Optional.

### Optionals and primitives

The regular Optional object can only hold object types, not primitives. To work with Optionals and primitives there are some variants on the Option class, namely:

- OptionalInt
- OptionalDouble
- OptionalLong

They have the same methods as the regular Optional, with some exceptions:
- The .get() method of Optional becomes respectively .getAsInt(), .getAsDouble() and .getAsLong().
- The parameter types of .orElseGet() are respectively IntSupplier, DoubleSupplier and LongSupplier (instead of Supplier).

One other point:
- The return type of IntStream.average(), DoubleStream.average() and LongSTream.average() is OptionalDouble. Note that .average() doesn't exist in the regular Stream class.
