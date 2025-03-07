## Removing from a list in a loop

In the book I encountered a paragraph (chapter 14) explaining that Java doesn't allow removing elements from a list while using the enhanced for loop:

```
List<String> list = new ArrayList<>(List.of("Aap","Noot","Mies"));

for (String s: list) {

    list.remove(s); -- throws ConcurrentModificationException, RunTimeException that makes program crash
}
```

This seems logical. The code compiles but a RuntimeException occurs. I tried it on a set and this gave the same runtime exception. I was wondering what would happen in a traditional for loop:

```
List<String> list = new ArrayList<>(List.of("Aap","Noot","Mies"));

for (int i=0; i<list.size(); i++){

    list.remove(i);
    System.out.println(list);
}

-- result:
-- [Noot, Mies]
-- [Noot]
```

I was expecting another RunTimeException, more precisely an IndexOutOfBoundsException, which is what happens when trying to apply a remove with an index number that's too large.  But no, the code compiles and runs without warning or exception. The loop loops two times instead of three, because list.size() decreases with every removal.

When doing it a bit differently, still no error: 

```
List<String> list = new ArrayList<>(List.of("Aap","Noot","Mies"));

for (int i=0; i<list.size(); i++){

    list.remove(0);
    System.out.println(list);
}

-- [Noot, Mies]
-- [Mies]
```

Because the size of the list shrinks, the iteration stops after the second iteration, which means that the index is never out of bounds. But that would mean that if list.size() is replaced by a literal, an IndexOutOfBoundsException should occur and indeed it does:

```
for (int i=0; i< 3; i++){

    list.remove(i);
    System.out.println(list);
}
-- IndexOutOfBoundsException during RunTime
```

My initial thought on this was that it wouldn't be possible to remove from a list in a regular for loop. In the past I learned about the Iterator interface, creating an iterator for a list which is possible with list.iterator(). The special quality of an Iterator object is that you can remove list elements during iteration. And while Iterator's cursor can only move forward, there happens to be the ListIterator object that can move forward and backward, and while ListIterator has remove(), ListIterator has add() as well.

Anyway, I'm not sure if ListIterator, available since Java 1.2, is still being used. Streams have taken over the world. My take on removing list elements during iteration is that:

- the enhanced for loop can't do it, it compiles and then crashes with ConcurrentModificationException.
- the regular for-loop can do it, as long as you find a way not to get out of bounds with the index. The number of cycles decreases every time you remove an element unless you use a literal (i<5 instead of i<list.size()).
- Iterator can do it, and ListIterator is an Iterator with more options (going backwards, adding elements).
- Streams can do it with filter(), no problem here because the streamed collection is unmodified, you are just creating a new collection with collect(Collectors.toList()).


 



