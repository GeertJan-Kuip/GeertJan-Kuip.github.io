## Using the command line for compiling, running the program and jar operations

Chapter one of the [Complete Study Guide](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1) for 1Z0-819 is about Java basics, with explanations on how to use the command line, how to write a file with a class containing main, how comments work and how to do import- and package declaration. It is all very basic but because I never use the command line (Intellij has buttons and menus for that) I learned a lot anyway.

After studying it my score on the 20 questions at the end of the chapter was 65%, and I can definitely improve on that if I get used to using the terminal for compiling, running and packaging stuff.

#### Sidenote: something about comments
This was part of chapter 1. They might ask about code with comments, whether it compiles or not. I learned that this pattern:

```
/**  */ 
```

is called Javadoc multi-line comment. It is what you see in the source files. One question in the chapter asked whether comments would compilate and I was tricked by this one, which (of course) doesn't compile. Remember, /* and */ are not anaolgous to brackets.

```
/*
* /*ferret*/
*/
```

#### java and javac command

The ```javac``` command compiles the code and the ```java``` command runs the code. The following lines compile and run a single .java file which has to have a main function.

```
javac MyClass.java

java MyClass
```

The first line creates the .class file, which is by default stored in the same folder as the .java file. The second line calls MyClass, no .class extension is to be used. I tried this in a project with a Maven folder structure and it didn't actually work. Compiling went alright with the following command (utilities is the name of a package, the Printing.class file was created in the utilities folder/package):

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java\utilities> javac Printing.java
```

What did not work was this:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java\utilities> java Printing
Caused by: java.lang.NoClassDefFoundError: Printing (wrong name: utilities/Printing)
```

To make it work I did this:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java\utilities> java -cp . Printing.java
Hi
```

By adding -cp (-classpath and --class-path would also work) and a . , which denotes the current directory, and .java, it worked. But actually this solution did not rely on the .class file I created earlier, as this is an example of a **single-file source-code command**, which can be applied when only one class without dependencies is used. It is a sort of hack whereby no .class file is used or generated and the bytecode of the Printing file is stored in memory. So this actually works as well:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java\utilities> java Printing.java
Hi
```

Anyway, I will have to find out why exactly things are not exactly working as I would like. It might have to do with the folder structure or the fact that I am working in a larger project with multiple files. 

Update: tried a bit more, went one level down in the source tree and this worked:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java> java utilities/Printing
Hi
```

_Next update (written 3 hours later): I start to understand what a classpath is. I am now able to create compiled files anywhere in my pc folder structure and make them run. The crux of it all is that you need to think of the full path to the .class file, and split that path in two. The first part is the part that you describe with the -d or the -cp (-classpath, --class-path) thing, the second one is what you describe with dots between the folders. This latter one is called the FQN, the Fully Qualified Name. It is the name of the package as found on top of the file extended by .\<classname\>. That FQN should not depend on where in your pc your project resides. The last folder of the class path is the source folder, and it can vary depending on where things are stored._


#### Compiling packages

Compile a package requires the following command, the code below should work:

```
javac SomePackageName/MyClass.java   --compiles one package with one file

javac SomePackageName/*.java SomeOtherPackageName/*.java  --compiles two packages, all .java files in them will be included
```

I tried the following in my own dummy project and it did work:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java> javac utilities/*.java calculation/*.java
```

For every .java file a .class file was generated in the same folder as the .java file. Better would it be to have it in another folder, there is a -d command for that:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java> javac -d compiledstuff utilities/*.java calculation/*.java
```

The -d compiledstuff creates a new directory named compiledstuff in the java folder. In this compiledstuff folder the folder tree of the project is copied, so you have folders in it with the two package names (utilities and calculation) and in these folders the class files are stored. To run it, do:

```
PS C:\Kuips files\Java projects\working-with-modules\module1\src\main\java> java -cp ./compiledstuff utilities.Printing
```

Note that it is important to separate the classpath from the FQN. The classpath is C:\Kuips files\Java projects\working-with-modules\module1\src\main\java\compiledstuff, the FQN is utilities.Printing. The latter corresponds with the declaration on top of Printing.java.


#### Compiling from scattered files

If the compiled .class files that you need for the program to run reside in different locations, you can do the following:

```
java -cp ".;C:\MyFolder\anotherfolder\*.java; C:\MyFolder\temp\myJar.jar" myPackage.MyClassWithMainInIt
```

Locations are separated by semicolon (MacOS and Linux have a different separator) and . represents the current directory. You can add jar files, java will look inside them and find all class files. The last part denotes the FQN of the file that contains main.

-- jar -cvf


#### Compiling with JAR files

To create a jar file from all files in the current directory, use one of the following commands:

```
jar -cvf myNewFile.jar

jar --create --verbose --file myNewFile.jar
```

Okay, it didn't work. On Stackoverflow I found [this post](https://stackoverflow.com/questions/29180639/java-jar-is-not-recognized-as-an-internal-or-external-command) and it suggested that I needed to add an environment variable to 'path' in the windows settings. I added 'C:\Program Files\Java\jdk1.8.0_40\bin\'. Still doesn't work, but after restarting the jar command was recognized.

Nevertheless, still no jar file because of another error that says I need to specify the files to include. The following commands works when I am in a folder with class files in it (Note that I omitted \--verbose):

```
jar --create --file myJar.jar *.class
```

If I'm in a file with .java files it works as well:

```
jar --create --file myJar2.jar *.java
```

Accidently I typed this and it worked as well but it had nothing to do with Java:

```
jar --create --file myJar2.jar "C:\Kuips files\temp"
```` 

It took all the files in "C:\Kuips files\temp", compressed them and made a .jar file of it. This jar file was put in the current directory. It means that the jar command is purely about zipping up content, no matter the file types. It means that this should work as well:

```
jar --create --file myJar2.jar "C:\Kuips files\\Java projects\working-with-modules\module1\src\main\java"
```

It does, but it gives a lot of nested folders starting with "Kuips files".

Anyway, let this be the start of me becoming a virtuoso with the command line. Oracle's Java tutorials have good material about the ins and outs of the command line and packaging programs, including modules. More to follow.




















