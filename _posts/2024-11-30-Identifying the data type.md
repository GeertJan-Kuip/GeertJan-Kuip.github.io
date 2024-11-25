##Identifying data types

From my past programming stints I remember that not knowing the datatypes you work with is one of the best ways to waste your time.

Datatypes are not difficult because there are so many of them, or because every language has its own. The problem is that it requires more than a bit of skill to figure out the datatype of a specific variable.

Yesterday, in Javascript, I tried to write a function that returns whether the argument is an object or an array. A logical step would be:

```typeof myVar === 'object'```

No way this works. In Javascript, not only objects, but also arrays, functions and 'null' return true on this statement. 

For me, being misinformed about datatypes has often resulted in senseless trial and error, using one inappropriate method after the other until the code finally works.

At the end the code works, but you don't know why and you learned nothing. 