## Concurrency - concurrent collections

Concurrent collections are required when you want to modify collections in a multithreaded environment. You can use non-concurrent interfaces to access these concurrent collections, or you can use one of their concurrent counterparts.

The naming of the concurrent collections (there are 7 that needs to be learned) have two patterns that sound unfamiliar but in fact they aren't. The first is 'SkipList', which can be translated to 'Tree'. ConcurrentSkipListSet and ConcurrentSkipListMap are equivalents of TreeSet and TreeMap. They sort themselves. 

The second is 'CopyOnWrite'. This indicates that on every addition, removal or modification (in the sense that the reference of an element is changed), a copy of the collection is made to a new underlying structure. The reference to the collection object does not change. The consequence hereof is that any iterator established prior to modification will not see the changes and will iterate over the original elements. Adding or removing elements during a loop will not have effect on the number of loops. The iterated content is decided at the start and won't change.

Note that a lot of memory is required in a situation where you write to CopyOnWrite collections, as every modification requires a new copy.

Below are the seven collections from the Concurrent API that are part of the test:

|Class name|Java Collections Framework interfaces|Elements ordered|Sorted|Blocking|
|----|----|----|----|----|
|ConcurrentHashMap|ConcurrentMap|No|No|No|
|ConcurrentLinkedQueue|Queue|Yes|No|No|
|ConcurrentSkipListMap|ConcurrentMap SortedMap NavigableMap|Yes|Yes|No|
|ConcurrentSkipListSet|SortedSet NavigableSet|Yes|Yes|No|
|CopyOnWriteArrayList|List|Yes|No|No|
|CopyOnWriteArraySet|Set|No|No|No|
|LinkedBlockingQueue|BlockingQueue|Yes|No|Yes|


### Class LinkedBlockingQueue and interface BlockingQueue

LinkedBlockingQueue implements the BlockingQueue interface, which is a subinterface of Queue. It thus has all the Queue methods plus two overloaded methods for offer() and poll():

|Method|Description|
|-----|-----|
|**_offer_**(E e, long timeout, TimeUnit unit)|Adds item to queue, waiting specified time and returning flase if time elapses before space is available|
|**_poll_**(long timeout, TimeOut unit)|Retrieves and removes from queue, waiting specified time and returning null if time elapses before the item is available|

Methods related to blockingQueue can throw InterruptedException so you need to catch them. The book does that with a try-catch block.

### Converting non-synchronous collections to synchronous

Instead of using the concurrent collection classes you can convert regular collections into synchronized collections via static methods of the Collections class. If you know beforehand that concurrency will be an item it is better to create a concurrent class from the start, but in some cases this conversion might be usefull.

The difference between these converted classes and the concurrency API collections is that you cannot use an enhanced loop and then modify them within that loop. If you do, you get a ConcurrentModificationException, just like normal collections would. For all other things, these converted collections are thread safe.

A list of all the static methods in Collections that create thread safe variants (the return values are the thread safe collections:

|       |
|------------|
|synchronizedCollection(Collection<T> c)|
|synchronizedList(List<T> list)|
|synchronizedMap(Map<K,V> m)|
|synchronizedNavigableMap(NavigableMap<K,V> m)|
|synchronizedNavigableSet(NavigableSet<T> s)|
|synchronizedSet(Set<T> s)|
|synchronizedSortedMap(SortedMap<K,V> m)|
|synchronizedSortedSet(SortedSet<T> s)|





