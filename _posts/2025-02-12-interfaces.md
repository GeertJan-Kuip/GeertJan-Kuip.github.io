## Interfaces 

I thought that interfaces didn't need more than abstract methods and that they actually could contain only that. The most important fact about interfaces is that you can make some interface the type of a variable, thus creating code with less dependencies and better maintainability.

While preparing for 1Z0-819 I learned that interfaces can inherit from multiple other interfaces using the keyword 'extends'. You can declare final abstract fields and methods within interfaces, and if you forget to write the static and/or final modifier ahead of a member variable it will add them during compilation without error. If you forget to put the static modifier ahead of a method, it will protest as non-static methods cannot have a body and should be abstract. Then there is the option to include default methods, which must have a body, and these default methods can be either ignored, overridden or used by implementing classes. Conflicts can arise when an interface inherits multiple default methods or a default method and an abstract method with the same signature. Conflicts can also arise when a class implements multiple interfaces that have default methods with the same name/signature. The conflict might also arise when one interface provide a default method and the other an abstract method and those have the same signature, I'm not yet sure about that.

Anyway, here's a bit of code that compiles:

```
package package01;

public interface MotherOfAllPrinting {

    default String print(String s){

        System.out.printf("This will be printed: %s%n", s);
        return s;
    }
}
```
```
package package01;

public interface FatherOfAllPrinting {

    String print(String s);

    Boolean doYouLikeToPrint();
}
```
```
package package01;

public interface Printable extends MotherOfAllPrinting, FatherOfAllPrinting{

    // this field will be made static and final automatically
    String newString = "Some static string variable in an interface";

    // static methods are allowed, you cannot omit static modifier
    static void sayHello(){
        System.out.println("Hello!\n");
    }

    @Override
    default String print(String s) {
        return (String.format("%s - %s%n",newString,s));
    }
}
```
```
import package01.Printable;

public class DocumentHandler implements Printable {

    public static void main(String... args){

        Printable.sayHello();
        Printable d = new DocumentHandler();
        String myString = d.print("my addition");
        System.out.println(myString);
        String answer = d.doYouLikeToPrint().toString();
        System.out.println("Do you like to print? " + answer);
    }

    @Override
    public Boolean doYouLikeToPrint() {
        return true;
    }
}
```
output:
```
Hello!

Some static string variable in an interface - my addition

Do you like to print? true
```
