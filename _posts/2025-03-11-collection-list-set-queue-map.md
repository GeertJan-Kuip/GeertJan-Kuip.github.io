## Collection, List, Set, Queue, Map

In an earlier post I classified collections based on their abstract superclasses. The book classifies them differently, based on the implemented interfaces. This is an overview of the methods defined in these interfaces that might be on the test.

### Collection interface

Collection defines the following methods. They are inherited by ArrayList, LinkedList, HashSet and TreeSet but not HashMap and TreeMap.

#### boolean *add*(E element)

Adds an element and returns true if successful, false if not. The latter can happen with sets.

#### boolean *remove*(Object object)

Removes the given object. True on successful removal, false on unsuccessful attempt. If int is supplied, it removes element by index, which is only possible for lists. To signify Integer instead of int, use Integer.valueOf(number).

#### boolean *isEmpty*()

Just as it says.

#### int *size*()

Same.

#### void *clear*()

\-

#### boolean *contains*(Object o)

Uses equals() to test.

#### boolean *removeIf*(Predicate\<? super E\> filter)

Removes all elements that are tested positive by the lambda.

#### void *forEach*(Consumer\<? super T\> action)

\-

### List interface

These six methods are implemented in ArrayList and LinkedList. Their relationship to lists can often be derived from the fact that they access elements by index. In sets, both TreeSet and HashSet, this is not possible.

#### boolean *add*(E element)

Adds element to the end of the list, returns true if successful.

#### void *add*(int index, E element)

Adds element at the specified position.

#### E *get*(int index)

Returns value at the given index.

#### E *remove*(int index)

Removes element at given index.

#### void *replaceAll*(UnaryOperator\<E\> op)

Replaces each element with the result of the operator.

#### E *set*(int index, E e)

Replaces the element at the index position with the new element. Returns the old element.

### Set interface

Only two methods from the Set interface need to be learned, namely .of and .ofCopy. They are the equivalent of List.of and List.ofCopy and return immutable sets.

### Queue interface

Queue is FIFO, first in first out. When you add an element, it will be placed at the end of the list (visually I would say on the right). When you retrieve an element from the collection, or when you peek at it, it will be the leftmost element (that went in first).

There are three methods that throw an exception if they cannot perform their intended action (for example when the queue is empty). There are three equivalent methods that perform the same actions but do not throw an exception if unsuccessful.

#### boolean *add*(E e)

Adds element to the end (right side) of collection. Throws exception if not possible.

#### E *element*()

Returns the element at the left of the collection without removing it. Throws exception if queue is empty.

#### E *remove*()

Returns the element at the left of the collection and removes it. Throws exception if queue is empty.

#### boolean *offer*(E e)

Similar to add, returns true if successful and false if not.

#### E *peek*()

Similar to element, returns null on empty queue.

#### E *poll*()

Similar to remove, returns null on empty queue.

### Map interface

Map objects do not implement Collection interface. Instead they get their methods from the map interface. There are quite a lot of methods to learn for it. Some of these methods have equivalents in Collection, they are a translation to key-value pairs. Others are just typical for maps.

### Creating maps

#### Map.*of*()

Equivalent of List.of and Set.of. Supply keys and values as arguments, (key1, value1, key2, value2).

#### Map.*copyOf*()

Equivalent of List.copyOf and Set.copyOf.

#### Map.*ofEntries*()

Requires entry objects to construct a map, like this:

``` 
Map.ofEntries(Map.entry(1,”Fish”), Map.entry(2,”Bird”), Map.entry(3, “Mammal”));  
```

Note that Map.entry() is a static method that returns a Map.Entry (uppercase vs lowercase).

### Methods identical to those of Collection interface

Map has 4 methods that are really not different from those of Collection, namely:

#### int *size*()

#### void *isEmpty*()

#### void *clear*()

#### void *forEach*()

Here forEach has a BiConsumer argument instead of a Consumer.

### Methods quite similar to those of Collection interface

#### V *put*(K key, K value)

Similar to add, but will replace key value pair if key already exists. Returns null if no overwrite or old value if overwrite/replace. This method is also similar to the set method of the List interface.

#### V *putIfAbsent*(K key, V value)

Same as put but does not replace pair if key already exists. Returns null if pair can be added and null if not.

#### V *remove*(Object key)

Removes entry if applicable and returns value mapped to key. If no match is found returns null. Note: there is no method similar to removeIf() in the Map interface.

#### boolean *containsKey*(Object key)

Returns whether key is in map.

#### boolean *containsValue*(Object value)

Returns whether value is in map.

### Methods similar to those of List interface

#### V *get*(Object key)

Returns the value mapped to key or null if key not present. Similar to List get method that has index as argument.

#### V *getOrDefault*(Object key, V defaultValue)

Same as previous but returns defaultValue instead of null if key not present.

#### void *replaceAll*(BiFunction\<K,V,V\> func)

While the replaceAll method in List has a UnaryOperator, this one has a BiFunction.

### Methods typical for Map interface

#### Collection\<V\> *values*()

Returns collection with all values.

#### Set\<K\> *keySet*()

Returns set containing all keys. Keys are unique so set is appropriate.

#### Set\<Map.Entry\<K,V\>\> *entrySet*()

Returns set with Entries.

#### V *merge*(K key, V value, Function(\<V,V,V\> func))

This is by far the most complicated one, don’t know yet how it behaves.