## LinkedList and more

LinkedList implements the List interface and thus has all its methods. Unlike ArrayList, it implements the Deque and Queue interface (Deque extends the Queue interface). This makes LinkedList suited for the role of queue and stack. ArrayList implements the RandomAcces, which LinkedList doesn't.

Trying to learn as much as possible about the implementation of collections in Java. There are a great many, some of them rather obscure (JobStateReason, ConcurrentLinkDeque, PriorityBlockingQueue). The main ones to know are simple Array, ArrayList, LinkedList, HashSet, TreeSet, HashMap and TreeMap.

To see the relationships you can look at parent classes or at implemented interfaces and their parent interfaces. The interfaces structure is harder to map than the class structure. Looking at parent classes shows where and how the implementations are being done and when and where they are overridden. For now I focus on the (easier) class-inheritance relationships.

### Simple Array

The name is just array but I found it confusing because of the existence of ArrayList. Simple array does not have its implementation files in the java.util package. It resides somewhere deeper in the code more close to the operating system. Simple array can be traversed with forEach and there are no methods associated with it other than those it inherits from Object. These methods (clone(), toString(0 etc) have nothing to do with array operations.

Simple arrays have a fixed size that is set on creation. They can be multidimensional. You can store primitives, objects and collections in it, which is not possible with higher order collections that do not store unwrapped primitives. Null is not allowed. Simple array is the basis for class ArrayList, that puts a wrapper around so the size becomes adjustable. Initializing an array can be done in different ways, some rather unexpected. Below some examples that compile well:

```
int  []myArray1 = new int[3];
int[]  myArray2 = new int[] {1,2,3};
int myArray3  [] = {1,2,3,};

int[][] myMultiArray1 = new int[2][2];
int  []myMultiArray2 [] = new int[2][];
int myMultiArray3[][] = {{1,2},{3,4}};
int[]  myMultiArray4[] = new int[8%3][3-1];
```

Setting and getting values is rather straightforward, and length is a property.

```
int a = myArray[2];
myArray[2] = 27;
int length = myArray.length;
```

### ArrayList, LinkedList, HashSet and TreeSet

I put these together because they have a common ancestor namely class AbstractCollection. The tree looks like this (I only included classes parent classes of the four types I studied):

* AbstractCollection
    * AbstractList
        * AbstractSequentialList
            * LinkedList
    * AbstractSet
        * HashSet
        * TreeSet








