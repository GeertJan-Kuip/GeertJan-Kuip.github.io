## Enums are classes (sort of)

In a previous blog I wrote about enums but ended up without knowing how to implement default methods for enum values. I know how it works now, and here I'll try to explain enums by showing how similar they are to classes. Many of enum's peculiarities become easier to grasp then. I'll do this by code examples, from very simple to more complex.

### Simple form

The enums below are the three simplest I can think of. Like classes, if you omit the access modifier, it will be package-private by default.

```
enum Cars{}  // this compiles even though it is empty

enum Cars{OPEL} // only one value

enum Cars{OPEL, FORD;}  // ; is optional as long as no parameters/arguments are provided
```

### With fields

When adding fields, the declarations of the constants start to look like object initializers but without the _new_ keyword. The parameters in the initializer need to be matched with a field and a constructor:

```
enum Cars{
    OPEL(155), FORD(170);
    int speed;
    private Cars(int speed){  // private is the only access modifier allowed
        this.speed=speed;
    }
}

System.out.println(Cars.OPEL.speed);  // 155
```

Important is to look at the implicit access modifiers. Constructors are implicitly private so you can omit the access modifier. This is different from classes where constructors are package-private by default. Fields are package-private by default, both in classes and enums. It is a good idea to make them private and use a get method. It is, if the value shouldn't change, also good to make them final.

```
enum Cars{
    OPEL(155), FORD(170);
    private final int speed;
    Cars(int speed){
        this.speed=speed;
    }

    public int getSpeed(){ return this.speed;} 
}

System.out.println(Cars.OPEL.getSpeed()); // 155
```

Note that speed isn't accessible anymore so I used getSpeed() instead. I omitted the private modifier for the constructor which is okay, it remains private.

In classes, constructors can be overloaded. The same is true for enums:

```
enum Cars{
    OPEL, FORD(170);
    private int speed;

    Cars(int speed){
        this.speed=speed;
    }

    Cars(){}

    public int getSpeed(){ return this.speed;}
}

System.out.println(Cars.OPEL.getSpeed()); // 0
```

The example above has two constructors. If OPEL is created, the constructor with 0 parameters is used, for FORD the other. To make it work, the final modifier for speed had to be removed. The speed of the OPEL instance is zero. Instead of ```OPEL, FORD(170);``` we could have written ```OPEL(), FORD(170);```. The latter better indicates the absence of construction arguments for OPEL.

### With methods

Construction of the enum objects takes place within the enum itself using a private constructor. Whe creating an object, whether enum or class, it is possible to override methods or to add methods, unless those methods are final or the class is final. Short example in which methods are added:

```
enum Cars{
    OPEL(){public String getColor(){ return "red";}},
    FORD(){private String getColor(){ return "yellow";}};  // COMPILE ERROR!

    abstract String getColor();

    Cars(){} // can be omitted here because it is identical to the default constructor
}

System.out.println(Cars.OPEL.getColor());  // red (if line 3 is adjusted)
```

The example above shows how you can implement methods. An abstract method is declared, whereby the abstract keyword is mandatory. The enum objects implement the abstract method and they are required to do so, just like it would be with regular classes that extend an abstract class. 

The reason that line 3 doesn't compile is that the private access modifier is more restrictive than the implicit package-private modifier of the abstract class. This is forbidden in general in Java.

That the abstract class has package-private as default is to be expected: in classes (and in enums) all fields and methods (except the constructors in enums) have package-private as default access modifier. This is unlike interfaces and annotations, whose methods and fields are public by default.

Question: what is the default access modifier for classes, enums, interfaces, and annotations? The answer is that they are all package-private. 

As enums are package-private by default, it might be necessary to make them public, depending on the way they will be used.

### Using default method

The sample below shows how to implement a default method. Warning: don't use the default keyword, that is not allowed.

```
enum Cars{
    OPEL(),
    FORD(){String getColor(){ return "yellow";}}; // default method gets overridden here

    String getColor(){
        return "red";
    };
}

System.out.println(Cars.FORD.getColor()); //yellow
```

If we would have replaced FORD with OPEL the printed value would be 'red'.

### Enum objects are created only once

Try to predict what the output of the following snippet will be:

```
enum Fuel{PETROL, DIESEL}

enum Cars{
    OPEL(180),
    FORD(){Fuel getFuel(){
        System.out.println("Ford's fuel method");
        return Fuel.DIESEL;
    }};

    private int speed;

    Fuel getFuel(){
        System.out.println("The default fuel method");
        return Fuel.PETROL;
    }

    Cars(){
        System.out.println("Constructor 1");
    }

    Cars(int speed){
        this.speed=speed;
        System.out.println("Constructor 2");
    }
}

System.out.println(Cars.FORD.getFuel());

// output:
"Constructor 2"
"Constructor 1"
"Ford's fuel method"
"DIESEL"
```

Okay, the answer was already included. But there are a few important mechanisms:

- upon calling the first enum, an enum instance of all the values is constructed in the order in which they are declared. As OPEL relies on the second constructor, "Constructor 2" is printed first and then "Constructor 1", even though we do call FORD and not OPEL.
- subsequent calls to enum values do not activate the constructors anymore, every construction activity is only done once.
- calling the getFuel method on FORD activates the method override implemented by Ford
- the last line printed is "DIESEL", as that is Ford's fuel. Note that it doesn't print "Fuel.DIESEL". The toString() of Fuel.DIESEL is just DIESEL.

### Few last things

Enum has some instance methods (**_name()_**, **_ordinal()_**, **_toString()_**, **_compareTo()_**, **_equals()_** and **_hasCode()_**). Note that the last four stem from Object(), which illustrates once again that enums are classes. Btw **_name()_** and **_toString()_** give the same return value ("DIESEL"). **_Ordinal()_** returns the position of a value in the declaration.

Static methods are **_values()_**, which returns an array with the enum values, and **_valueOf()_**. The latter can be used like this:

```
Fuel fuel = Fuel.valueOf("DIESEL");  // DIESEL
```

This snippet gives runtime error if the String argument (case sensitive) does not exist in the enum. The error is IllegalArgumentException. It is not a compile error, as the compiler does not evaluate strings.

Last thing: enum values are written snake case by convention but this is not mandatory.

Last thing 2: in switch-case statements/expressions you must use DIESEL, Fuel.DIESEL doesn't compile: 

```
Fuel fuel = Fuel.DIESEL;

switch(fuel){
    case DIESEL:
        System.out.println("Yes");
        break;
    case PETROL:
        System.out.println("Other");
        break;
}
```

PS Do not forget the break statement in switch statements/expressions.






