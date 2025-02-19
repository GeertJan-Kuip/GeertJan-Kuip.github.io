## LinkedList and more 

LinkedList implements the List interface and thus has all its methods. Unlike ArrayList, it implements the Deque and Queue interface (Deque extends the Queue interface). This makes LinkedList suited for the role of queue and stack. ArrayList implements the RandomAcces, which LinkedList doesn't.

Trying to learn as much as possible about the implementation of collections in Java. There are a great many, some of them rather obscure (JobStateReason, ConcurrentLinkDeque, PriorityBlockingQueue). The main ones to know are simple Array, ArrayList, LinkedList, HashSet, TreeSet, HashMap and TreeMap.

To see the relationships you can look at parent classes or at implemented interfaces and their parent interfaces. The interfaces structure is harder to map than the class structure. Looking at parent classes shows where and how the implementations are being done and when and where they are overridden. For now I focus on the (easier) class-inheritance relationships.

### Simple Array

The name is just array but I found this confusing because of the existence of ArrayList. Simple array does not have its implementation files in the java.util package. It resides somewhere deeper in the code more close to the operating system. Simple array can be traversed with forEach and there are no methods associated with it other than those it inherits from Object. These methods (clone(), toString(0 etc) have nothing to do with array operations.

Simple arrays have a fixed size that is set on creation. They can be multidimensional. You can store primitives, objects and collections in it, which is not possible with higher order collections that do not store unwrapped primitives. Null is allowed. Simple array is the basis for class ArrayList, that puts a wrapper around so the size becomes adjustable. Initializing an array can be done in different ways, some rather unexpected. Setting and getting values is rather straightforward, and length is a property.

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
        * ArrayList
    * AbstractSet
        * HashSet
        * TreeSet

To see what classes have in common, we need to know what is in AbstractCollection, AbstractList, AbstractSequentialList and AbstractSet. Writing them down works therapeutically, so here are the method signatures of each of them (I omitted the types of the return values). It is important to note that while the abstract classes have a lot of methods implemented, some of them actually have a method body that does nothing more than 'throw new UnsupportedOperationException()'. These methods are overridden on lower levels of the lineage tree.


#### AbstractCollection

Abstract methods:
- iterator();
- size();

Concrete methods:
- add(E e);
- addAll(Collection<? extends E> c);
- clear();
- contains(Object o);
- containsAll(Collection<?> C);
- isEmpty();
- remove(Object o);
- removeAll(Collection<?> c);
- retainAll(Collection<?> c);
- toArray();
- toArray(T[] a);
- toString();


#### AbstractList

Fields
- modCount

Abstract methods
- get(int index);

Concrete methods
- add(E e);
- add(int index, E element);
- addAll(int index, Collection<? extends E> c);
- clear();
- equals(Object o);
- hasCode();
- indexOf(Object o);
- iterator();
- lastIndexOf(Object o);
- listIterator();
- listIterator(int index);
- remove(int index);
- removeRange(int fromIndex, int toIndex);
- set(int index, E element);
- subList(int fromIndex, int toIndex);


#### AbstractSequentialList

Abstract methods
- listIterator(int index);

Concrete methods
- add(int index, E element);
- addAll(int index, Collection<? extends E> c);
- get(int index);
- iterator();
- remove(int index);
- set(int index, E element);


#### LinkedList

Fields
- modCount (inherited from AbstractList)

Concrete methods
- add(E e);
- add(int index, E element);
- addAll(Collection<? extends E> c);
- addAll(int index, Collection<? extends E> c);
- addFirst(E e);
- addLast(E e);
- clear();
- clone();
- contains(Object o);
- descendingIterator();
- element();
- get(int index);
- getFirst();
- getLast();
- indexOf(Object o);
- lastIndexOf(Object o);
- listIterator();
- offer(E e);
- offerFirst(E e);
- offerLast(E e);
- peek();
- peekFirst();
- peekLast();
- poll();
- pollFirst();
- pollLast();
- pop();
- push(E e);
- remove();
- remove(int index);
- remove(Object o);
- removeFirst();
- removeFirstOccurrence(Object o);
- removeLast();
- removeLastOccurrence(Object o);
- set(int index, E element);
- size();
- spliterator();
- toArray();
- toArray(T[] a);


#### ArrayList

fields
- modCount (inherited from AbstractList)

Concrete methods
- add(E e);
- add(int index, E element);
- addAll(Collection<? extends E> c);
- addAll(int index, Collection<? extends E> c);
- clear();
- clone();
- contains(Object o);
- ensureCapacity(intminCapacity);
- forEach(Consumer<? super E> action);
- get();
- indexOf(Object o);
- isEmpty();
- iterator();
- lastIndexOf(Object o);
- listIterator();
- listIterator(int index);
- remove(int index);
- remove(Object o);
- removeAll(Collection<?> c);
- removeIf(Predicate<? super E> filter);
- removeRange(int fromIndex, int toIndex);
- replaceAll(UnaryOperator<E> operator);
- retainAll(Collection<?> c);
- set(int index, E element);
- size();
- sort(Comparator<? super E> c);
- spliterator();
- subList(int fromIndex, int toIndex);
- toArray();
- toArray(T[] a);
- trimToSize();


#### AbstractSet

Concrete methods
- equals(Object o);
- hashCode();
- removeAll(Collection<?> c);


#### HashSet

Concrete methods
- add(E e);
- clear();
- clone();
- contains(Object o);
- isEmpty();
- iterator();
- remove(Object o);
- size();
- spliterator();


#### TreeSet

Concrete methods
- add(E e);
- addAll(Collection<? extends E> c);
- ceiling(E e);
- clear();
- clone();
- comparator();
- contains(Object o);
- descendingIterator();
- descendingSet();
- first();
- floor(E e);
- headSet(E toElement);
- headSet(E toElement, boolean inclusive);
- higher(E e);
- isEmpty();
- iterator();
- last();
- lower(E e);
- pollFirst();
- pollLast();
- remove(Object o);
- size();
- spliterator();
- subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive);
- subSet(E fromElement, E toElement);
- tailSet(E fromElement);
- tailSet(E fromElement, boolean inclusive);


### HashMap, TreeMap

HashMap and TreeMap both have AbstractMap as parent class. Their methods are the following:


#### AbstractMap

Abstract methods:
- entrySet();

Concrete methods:
- clear();
- clone();
- containsKey(Object key);
- containsValue(Objact value);
- equals(Object o);
- get(Object key);
- hashCode();
- isEmpty();
- keySet();
- put(K key, V value);
- putAll(Map<? extends K, ? extends V> m);
- remove(Object key);
- size();
- toString();
- values();


#### HashMap

Concrete methods:
- clear();
- clone();
- compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction);
- computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction);
- computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction);
- containsKey(Object key);
- containsValue(Object value);
- entrySet();
- forEach(BiConsumer<? super K,? super V> action);
- get(Object key);
- getOrDefault(Object key, V defaultValue);
- isEmpty();
- keySet();
- merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction);
- put(Key key, V value);
- putAll(Map<? extends K>, ? extends V> m);
- putIfAbsent(K key, V value);
- remove(Object key);
- remove(Object key, Object value);
- replace(K key, V value);
- replace(K key, V oldValue, V newValue);
- replaceAll(BiFunction<? super K,? super V,? extends V> function);
- size();
- values();


### TreeMap

Concrete methods:
- ceilingEntry(K key);
- ceilingKey(K key);
- clear();
- clone();
- comparator();
- containsKey(Object key);
- containsValue(Object value);
- descendingKeySet();
- descendingMap();
- entrySet();
- firstEntry();
- firstKey();
- floorEntry(K key);
- floorKey(K key);
- forEach(BiConsumer<? super K, ? super V> action);
- get(Object key);
- headMap(K toKey);
- headMap(K toKey, boolean inclusive);
- higherEntry(K key);
- higherKey(K key);
- keySet();
- lastEntry();
- lastKey();
- lowerEntry(K key);
- lowerKey(K key);
- navigableKeySet();
- pollFirstEntry();
- pollLastEntry();
- put(K key, V value);
- putAll(Map<? extends K, ? extends V> map);
- remove(Object key);
- replace(K key, V value);
- replace(K key, V oldValue, V newValue);
- replaceAll(BiFunction<? super K,? super V,? extends V> function);
- size();
- subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive);
- subMap(K fromKey, K toKey);
- tailMap(K fromKey);
- tailMap(K fromKey, boolean inclusive);
- values();


### Concluding

There are more collection types than the one mentioned here but these might be the ones most used. The way methods are inherited and implemented follows a rather Byzantine structure, there is logic in it but it takes a while to deconstruct. 

I'm rather familiar with ArrayMap and HashSet and I need to learn more about the others, especially TreeMap and TreeSet. The source code of LinkedList can be read quite easily and I now understand why this is the collection type for stacks, queues and dequeues. 

In general: collection types that have an order tend to have more methods available, and collection types that can be used as stack, queue or deque have more methods than collection types that can't. 





















