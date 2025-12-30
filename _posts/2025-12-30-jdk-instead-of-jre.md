# JDK instead of JRE

I'm working hard on a web app that lets you select a Java package on GitHub and translate it into a valid UML schema. As I want to go to the bottom of compiler internals, I need to some libraries that are available in the full Java Development Kit (jdk.compiler) but not in the Java Runtime Environment (JRE). I noticed this unavailability when Intellij didn't automatically provide the right imports for the Symbol class `com.sun.tools.javac.code.Symbol`. 

## Symbol is obscure

The UML schema needs to know about dependencies and I learned that finding dependencies can best be done via Symbols. A symbol is everything in your code that has a name, during compilation (task.analyze) Javac generates all sorts of information about all symbols and stores it in tables, objects etc. Class Symbol, plus all its derivatives like Symbol.ClassSymbol and Symbol.MethodSymbol, can be accessed via the Element interface, which is part of the JRE, but if you want to go deeper you need those JDK imports. Illustrative of the obscurity of the Symbol class is this Javadoc text in its source code:

_This is NOT part of any supported API. If you write code that depends on this, you do so at your own risk. This code and its internal interfaces are subject to change or deletion without notice._

## Why I need Symbol and its derivatives

Actually I do not know exactly yet. The best help for these compiler internals I have is AI and ChatGPT assured me that getting all the dependency relations right would be done most effectively using symbol information. I had first try to do it using type information but I'm happy to follow ChatGPT in this.

## Why I cannot rely on Intellij

Intellij is happy enough to adjust its environment for me so that I can use `com.sun.tools.javac` imports like these:

```
import com.sun.tools.javac.code.Symbol;
import com.sun.tools.javac.code.Symbol.ClassSymbol;
import com.sun.tools.javac.code.Symbol.MethodSymbol;
import com.sun.tools.javac.code.Symbol.VarSymbol;
import com.sun.tools.javac.code.Type;
```

To do so, Java must be started like this:

```
java \
  --add-modules jdk.compiler \
  --add-exports jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED \
  --add-exports jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED \
  --add-exports jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED \
  -jar your-app.jar
```

Once I added the right import statements to the class I worked on Intellij proposed to add these statements as _Run Configuration VM options_. I don't even know where these reside in the GUI. But as soon as you run the app in another environment, it won't work anymore. Intellij takes way too good care of me, at the cost of me being unaware of all the settings and adjustments it makes under the hood. I want these required settings to be very explicitly defined so I know how to make the app run outside if the Intellij environment.

## How to deal with it

ChatGPT suggested to make the --add-modules and --add-exports flags explicit. On the server in a [systemd unit file](https://geertjan-kuip.github.io/2025/09/10/systemd-units.html):

```
ExecStart=/usr/bin/java \
  --add-modules jdk.compiler \
  --add-exports jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED \
  --add-exports jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED \
  -jar /opt/app/app.jar
```

Or in a Docker file:

```
FROM eclipse-temurin:21-jdk
ENTRYPOINT ["java",
 "--add-modules", "jdk.compiler",
 "--add-exports", "jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED",
 "--add-exports", "jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED",
 "-jar", "/app/app.jar"]
```

## The best option

ChatGPT proposed a 'best' solution using jlink. I have no experience with it and do not know how to do it but it allows you to create a single package that does the whole job at once. You create a custom runtime image that includes the jdk.compiler, exports the needed packages and ships everything with your app. You deploy one directory:

```
my-app-runtime/
  bin/java
  lib/modules
  app.jar
```

And start it with:

```
./bin/java -jar app.jar
```

I will have to sort out if this is viable for me. 






