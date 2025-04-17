## Serialization and security

This is a topic I'm not very familiar with and serialization is not covered in-depth in the book, nor will it be on the upcoming exam. Nevertheless, serialization is explained in chapter 19 (I/O) and it is a subchapter in chapter 22 (Security). 

### Overview of serialization

Serialization is the process of converting an in-memory object to a byte stream. It's counterpart is deserialization. Some essentials:

- Only objects with the Serializable or Externalizable interface implemented can be serialized. Externalizable is a subinterface of Serializable. Serializable is a marker interface, Externalizable has two methods that must be implemented. Externalizable gives you extra control.
- Writing an object to a stream means writing its instance variables to the stream, it is not like the whole contents of the class.
- To communicate which instance variables are to be included, you can either mark those that you do not want to be included with _transient_, or you can create a special variable named _serialPersistentFields_ in your class. It has a special type and requires a specific initialization. It must be private static final otherwise it will be ignored. This is an example:

```
-- fields to be included
private final String name;
private final List<ClassicCar> collection;

private static final ObjectStreamField[] serialPersistentFields =
        {new ObjectStreamField("name", String.class),
                new ObjectStreamField("collection", List.class) };
```

- Static fields / class variables are never included.
- To write an object to a file you need to initialize an ObjectOutputStream object. It is best to use a try-with-resources construct. This is an example in which two Owner objects are created and written to a file:

```
File file  = new File("data.ser");

Owner peter = new Owner("Peter Jackson", c1, c2, c3, c4);
Owner jenny = new Owner("Jenny Rogers", c5, c6);

try(var out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(file)))){
    out.writeObject(peter);
    out.writeObject(jenny);
}
```

- It is required that every object referred to in the object you write to the file also has Serializable implemented. The standard Java classes (Integer, ArrayList etc.) meet this condition. There can be lots of recursion you need to be aware of, as the classes you refer to might have other classes that they refer to etc.
- The types of instance members that are null or that are marked transient do not have to meet this requirement. 
- One more thing: it is 'good practice' to declare a static serialVersionUID variable in every class that implements Serializable. This variable is stored with each object, and you get error if you try to 'read' an object from an inputstream into a class that has a different version id. This variable must be staticand final, just like serialPersistentFields. It is strongly recommended to make it private. The type must be 'long'. Example:

```
private static final long serialVersionUID = 10L;
```

### Reading an ObjectInputStream




### Serialization and security





