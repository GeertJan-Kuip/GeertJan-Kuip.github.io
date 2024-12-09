## Why null is an object in Javascript

Null in Javascript, I learned, is an object which means that

```typeof null;```

returns 'object'. I wondered about this, just as I wondered why typeof [1,2,3] returns 'object' as well. The most puzzling statement is this:

```typeof [1,2,3] === 'object'; //true```

What makes it puzzling is that objects in Javascript appear to be well defined datatypes, with own methods and properties, clearly different from thos of arrays. Null doesn't have any methods and properties at all.

What I found out is that object is a prototype of array, and that some obscure properties and methods of object are inherited by array. To array, child of mother object, is added an overwhelming list of properties and methods, among which a property that makes arrays iterable. 

Null on the other hand is an object as well but the properties and methods of object are taken away from it. I still don't know whether object is the parent of null or vice versa. Nevertheless, it is still an object, which means you can create a clean new object with

```myObject = Object.create(null);```

There's a lot more to say about it, for example that methods and properties have 4 flags that have to do with writability, enumerability etc. 

I'm happy to know about this, and I am poised to learn more about it.  



