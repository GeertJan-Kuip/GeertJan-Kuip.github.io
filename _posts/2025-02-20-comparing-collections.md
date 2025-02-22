## Comparing collections 

Below I created two tables comparing the different Java collection types. It is mainly relevant with regards to 1Z0-819.

The first table is about characteristics, the second about implemented interfaces.

|                      |      ArrayList       |    LinkedList    |       HashSet        |                      TreeSet                      |               HashMap                | TreeMap                                                                                                  |
|----------------------|:--------------------:|:----------------:|:--------------------:|:-------------------------------------------------:|:------------------------------------:|:--------------------------------------------------------------------------------------------------------:|
| resizable            |          ✅           |        ✅         |          ✅           |                         ✅                         |                  ✅                   | ✅                                                                                                        |
| duplicates allowed   |          ✅           |        ✅         |                      |                                                   |             ✅<br/>(not keys)              | ✅<br/>(not keys)                                                                                              |
| access by index      |          ✅           |        ✅         |                      |                                                   |                                      |                                                                                                          |
| auto-sorted          |                      |                  |                      |                         ✅                         |                                      | ✅                                                                                                        |
| fixed order          |          ✅           |        ✅         |                      |                                                   |                                      |                                                                                                          |
| synchronized         |                      |                  |                      |                                                   |                                      |                                                                                                          |
| null allowed         |          ✅           |        ✅         |     ✅<br/>just 1     |                                                   |    ✅<br/>(1 as key, multiple as value)    | ✅<br/>(only value, not key)                                                                                   |
|excels at| add/remove/ contains | stack operations | add/remove/ contains | find first, last, <br/>range queries, <br/>closest elements | get, put, lookup, <br/>large collections  | finding first, next, range queries, <br/>closest element, large collections, <br/>case-insensitive key comparisons |


-


|                   | ArrayList |     LinkedList     | HashSet  | TreeSet | HashMap | TreeMap |
|-------------------|:---------:|:-------:|:-----:|:-------:|:-------:|:-------:|
| Serializable      |     ✅     |         ✅          |    ✅     |    ✅    |    ✅    |    ✅    |
| Clonable          |     ✅     |         ✅          |    ✅     |    ✅    |    ✅    |    ✅    |
| Iterable<E>       |     ✅     |         ✅          |    ✅     |    ✅    |         |         |
| Collection<E>     |     ✅     |         ✅          |    ✅     |    ✅    |         |         |
| List<E>           |     ✅     |         ✅          |          |         |         |         |
| RandomAccess      |     ✅     |                    |          |         |         |         |
| Deque             |           |         ✅          |          |         |         |         |
| Queue             |           |         ✅          |          |         |         |         |
| Set<E>            |           |                    |    ✅     |    ✅    |         |         |
| NavigableSet<E>   |           |                    |          |    ✅    |         |         |
| SortedSet<E>      |           |                    |          |    ✅    |         |         |
| Map<K,V>          |           |                    |          |         |    ✅    |    ✅    |
| NavigableMap<K,V> |           |                    |          |         |         |    ✅    |
| SortedMap<K,V>    |           |                    |          |         |         |    ✅    |
