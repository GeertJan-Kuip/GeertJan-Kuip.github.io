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

### Deserialization

Code below is the reverse of code above. It deserializes a stream to get the objects back from the file:

```
List<Owner> owners = new ArrayList<>();

try(var in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(file)))){

    while(true){
        var obj = in.readObject();
        if (obj instanceof Owner) {
            owners.add((Owner) obj);
        }
    }
}catch(EOFException e){
    System.out.println("End of file.");
}
```

Few things to note:
- Objects need to be casted back to Owner. I know that type information is stored in the stream but I got a compile error when omitting the cast, and readObject() has Object as return type. See [documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/ObjectInputStream.html#readObject()).
- There is no way to know when the stream will end (like hasNext()) so you must create a catch with an EOFException.
- Every serializable object that was referenced to in the Owner object and not marked transient is included as well. In this case, Owner has an instance field of type List<ClassicCar>, which was included, and that collection of classic cars is fully exported and imported.

_Generally: all reading and writing requires handling IOExceptions. The .readObject() method throws a ClassNotFoundException as well._

From the book: 'For the exam, you need to understand how a deserialized object is created. When you deserialize an object, _the constructor of the serialized class, along with any instance initializers, is not called when the object is created._ Java will call the no-arg constructor of the first nonserializable parent class it can find in the class hierarchy. In our Gorilla class, this would be the no-arg constructor of Object.' 'Values that are not provided will be given their default Java value, such as null for String, or 0 for int values.'


### Serialization and security

The most straightforward way to serialize data without exposing secret information to the outer world is by using the transient modifier on fields you want to keep secret. You can as well use the serialPersistentFields variable to indicate which fields are to be serialized, as it forces you to explicitly whitelist the fields to be serialized (whitelisting is safer than blacklisting).

But when you take more control you can do other things as well, for example encrypt certain instance fields before they enter the stream.

#### Adding writeObject() and readObject()

To do so you need to add _writeObject()_ and _readObject()_ methods to your class. Although their names suggest that they are overrides of the readObject and writeObject methods of ObjectInputStream and ObjectOutputStream, they aren't. Their declarations are the following:

```
private void writeObject(ObjectOutputStream s) throws Exception{
//code
}

private void readObject(ObjectInputStream s) throws Exception{
//code
}
```

#### Bodies of writeObject() and readObject()

Perplexity assured me that these methods are not overrides but that the serialization system is checking for these specific signatures at runtime. If they are not provided, the default fallback for Java is ObjectOutputStream.defaultWriteObject() and ObjectInputStream.defaultReadObject(). 

Perplexity pointed to a [website](https://howtodoinjava.com/java/serialization/custom-serialization-readobject-writeobject/) with an example. In this example, the bodies of writeObject and readObject had the following content:

```
private void writeObject(ObjectOutputStream aOutputStream) throws IOException
{
    aOutputStream.writeUTF(firstName);  // writeUTF is a method from ObjectOutputStream
    aOutputStream.writeUTF(lastName);
    aOutputStream.writeInt(accountNumber);
    aOutputStream.writeLong(dateOpened.getTime());
}

private void readObject(ObjectInputStream aInputStream) throws ClassNotFoundException, IOException
{
    firstName = aInputStream.readUTF(); // readUTF is a method from ObjectInputStream
    lastName = aInputStream.readUTF();
    accountNumber = aInputStream.readInt();
    dateOpened = new Date(aInputStream.readLong());
}
```

The author stressed that the order in which the instance fields are written must be the same order in which they are read.

#### Better bodies: PutField and GetField

The book has a slightly different approach, using nested static classes of the ObjectOutputStream and ObjectInputStream classes, named PutField and GetField. Instances of them are created with respectively the putFields() and readFields() methods. I tend to think that the PutField instance works like a buffer or a to-do list in which you collect those things you want to add to the stream, and once you have filled it you call the writeFields() method. 

The GetField instance does something similar but reversed, it extracts all the instance fields from the stream and keeps them so you can process them. Creating the GetField instance by calling readFields on the ObjectInputStream immediately fills the GetField instance with all the relevant material. 

This is sample code from the book (chapter 22):

```
// getters, setters, constructors

private static final ObjectStreamField[] serialPersistentFields = 
    {new ObjectStreamField("name", String.class),
    new ObjectStreamField("ssn", String.class)};

private static String encrypt(String input){}

private static String decrypt(String input){}

private void writeObject(ObjectOutputStream s) throws Exception {
    ObjectOutputStream.PutField fields = s.putFields();
    fields.put("name", name);  // "name" is the instance variable name, name is the value that is written to the stream
    fields.put("ssn", encrypt(ssn)); // customization: variable ssn is written to the 
                                     // stream in encrypted form (encrypt is a custom function in the class).
    s.writeFields();
}

private void readObject(ObjectInputSTream s) throws Exception {
    ObjectInputStream.GetField fields = s.readFields();
    this.name = (String) fields.get("name", null);  // null is a value used when the stream does not have a value for it
    this.ssn = decrypt((String) fields.get("ssn", null));
}
```

I wondered about 'null' in ```this.name = (String) fields.get("name", null);``` but this is the value used if no other value available. See [Oracle documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/ObjectInputStream.GetField.html#get(java.lang.String,java.lang.Object)).

Btw encrypting and decrypting must not be done with passwords, store them only in encrypted form with salt and never decrypt them. 

Another btw: this example again used serialPersistentFields.

#### Why writeObject and readObject

It gives you more control about the form in which you export the instance variables. 









