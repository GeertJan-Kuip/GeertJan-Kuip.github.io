## Enthuware test result

To prepare for 1Z0-819 I bought access to the online mock tests of [Enthuware](https://enthuware.com/). I did a test from [Udemy](https://www.udemy.com/) as well. From both I like the convenience of doing it on online forms, which is better than the ebook pen and paper tests of the book I learn from.

For the Udemy test I scored 66% some days ago, for the first Enthuware test done this morning I scored 64%. Both are better than the scores I get for the tests from the book and I'm still below 68%. Work to do.

### Review

This is some things I learned from my wrong answers:

- The condition expression in an if statement can be a method call as long as the method returns a boolean. Of course it can, sill me. ```if(str.length()<5) {}```
- The statement ```if (false) ; else ; ;``` is legal. Both the if and the else clause can have empty statements. A semicolon signifies an empty statement.
- @SuppresWarnings and @Override are defined with @Retention(SOURCE).
- @SafeVarargs, @FunctionalInterface and @Deprecated are defined with @Retention(RUNTIME) 
- Map interface has methods _remove(Object o)_ and _values()_, not _contains(Object o)_.
- Conversion from int to float doesn't need a cast. Of course not.
- Conversion from short to char needs a cast and vice versa, because their ranges are not compatible.

- New thing: if a final variable whose value fits in a smaller type, you can assign it without a cast because the compiler already knows its value and concludes that it fits into the smaller type. It is called implicit narrowing and is allowed between byte, char, short and int but not float, double and long.

```
final int x = 100;
byte b = x; // alright

final long t = 100;
byte b2 =  t;  // compile error, cast required. Apparently only int works
```

- var not allowed for member fields of classes and inner/nested/local/anonymous classes, only within methods.
- Optional labels can be applied almost anywhere, there seems to be no clear restriction.
- The charAt() method can take a char value as an argument. It will be implicitly promoted to int.
- A module that uses a service must require the module that defines the service interface. It also must have a uses clause, like 'uses org.printservice.api.Print;
- ```while ( ) break ;``` is invalid as a condition expression in a while header is required.
- In a 'normal' if or else block you cannot use 'break' or 'continue', unless you place a loop within that block. One exception:

```
label: if(true){
    System.out.println("break label");
    break label; //this is valid because of the label
}
```

- Question: where are break, continue and return allowed and not allowed? When are curly brackets mandatory?
- module-info.java must be in the root folder of the packages.
- A static nested class can contain a non-static inner class.
- Local nested classes (inside a method) cannot be static.
- ```System.exit(0);``` closes the JVM immediately, nothing is done anymore, no closing of resources, no finally statements.
- When a JDBC connection is created, it is in auto-commit mode.

### Intermezzo

I copied explanation by Enthuware about modules and command line:

_"You need to know about three command line options for running a class that is contained in a module:

1. --module-path or -p: This option specifies the location(s) of the module(s) that are required for execution. This option is very versatile. You can specify exploded module directories, directories containing modular jars, or even specific modular or non-modular jars here. The path can be absolute or relative to the current directory. For example, --module-path c:/javatest/output/mathutils.jar or --module-path mathutils.jar

You can also specify the location where the module's files are located. For example, if your module is named abc.math.utils and this module is stored in c:\javatest\output, then you can use: --module-path c:/javatest/output.  Remember that c:\javatest\output directory must contain abc.math.utils directory and the module files (including module-info.class) must be present in their appropriate directory structure under abc.math.utils directory.

You can specify as many jar files or module locations separated by path separator (; on windows and : on  *nix) as required.

NOTE: -p is the short form for --module-path.(Observe the single and double dashes).

2. --module or -m: This option specifies the module that you want to run. For example, if you want to run abc.utils.Main class of abc.math.utils module, you should write --module abc.math.utils/abc.utils.Main
If a module jar specifies the Main-Class property its MANIFEST.MF file, you can omit the main class name from  --module option. For example, you can write, --module abc.math.utils instead of --module abc.math.utils/abc.utils.Main.

NOTE: -m is the short form for --module.(Observe the single and double dashes).

Thus,
java --module-path mathutils.jar --module abc.math.utils/abc.utils.Main is same as
java -p mathutils.jar -m abc.math.utils/abc.utils.Main

NOTE: It is possible to treat modular code as non-modular by ignoring module options altogether. For example, if you want to run the same class using the older classpath option, you can do it like this:
java -classpath mathutils.jar abc.utils.Main

3. -classpath: Remember that modular code cannot access code present on the -classpath but "automatic modules" are an exception to this rule. When a non-modular jar is put on --module-path, it becomes an "automatic module" but it can still access all the modular as well as non-modular code. In other words, a class from an automatic module can access classes present on --module-path (only the ones that are exported by the modules) as well as on -classpath without having any "requires" clause (remember that there is no module-info in automatic modules).
Thus, if your modular jar A depends on a non-modular jar B, you have to put that non-modular jar B on --module-path. You must also add appropriate requires clause in your module A's module-info, otherwise compilation of your module will not succeed. Further, if the non-modular jar B depends on another non-modular jar C, then the non-modular jar C may be put on the classpath or module-path."_

### End of intermezzo

- A CallableStatement is easier to build and call from JDBC code than a PreparedStatement. 
- ```&``` can have integral as well as boolean operands.
- lock() method returns void. tryLock() returns true or false. 
- tryLock​(long time, TimeUnit unit) throws true or false or an InterruptedException.
- Assigning a void return value to a variable gives compile error, like here:

```
Lock lock = new ReentrantLock();
var a = lock.lock();  // compile error
```

Last one:

What will the following code print?

```
public class TestClass{

   static char ch;
   static float f;
   static boolean bool;

   public static void main(String[] args){
      System.out.print(f);
      System.out.print(" ");
      System.out.print(ch);
      System.out.print(" ");
      System.out.print(bool);
   }
}
-- 0.0  false
```

- Printing a float doesn't give an 'F' as postfix (of course not, only used when assigning).
- char instance variable is by default initialized at 0. If printed, it prints the Null character Unicode U+0000. That is not a '0'.
