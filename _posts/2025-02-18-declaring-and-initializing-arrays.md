## Declaring and initializing arrays 

Here my notes on simple arrays. They lack the methods that higher order collections have but you can use static methods from the Arrays class to compensate for it. You can be creative when initializing an array, uninitialized variables are not permitted, and it's okay to use them to store the children of interfaces or abstract classes.


### On valid ways of initializing

All of the following statements compile:

```
{% raw %}
int  []myArray1 = new int[3];
int[]  myArray2 = new int[] {1,2,3};
int myArray3  [] = {1,2,3};

int[][] myMultiArray1 = new int[2][2];
int  []myMultiArray2 [] = new int[2][];
int myMultiArray3[][] = {{1,2},{3,4}};
int[]  myMultiArray4[] = new int[8%3][3-1];
```

An array with an uninitialized value doesn't compile:

```
String a;        
String[] stringArray = {"hi", a};
```

Having null in an array is okay and compiles:

```
String[] stringArray = {"hi", null};
{% endraw %}
```

An array can store primitives, object instances, abstract class types and interface types. An array for abstract class types can contain instances of all subclasses, while an array for interface types can contain instances of all implementing classes.

An array is an Object and can thus be cloned. If the array stores primitives, it will be a deep copy (values are copied). If the array contains reference values, then a shallow copy is created.

A multidimensional array is an array of objects, which are reference types. Cloning a multidimensional array means that it will be a shallow copy, with references to the same subarrays.

As an object, array has the equals() method. This never works. Use the static Arrays.equals() method instead.

