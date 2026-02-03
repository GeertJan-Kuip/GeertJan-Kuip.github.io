# Forbidden packages

While working on experiments with parsing I got encouraged by ChatGPT to use certain classes and packages that live in the jdk.compiler module. These packages are not exported or they are only exported to some specific other modules. In the JPMS (Java Platform Module System) with its strong encapsulation this means that you cannot import/use these classes in your own code.

Fortunately there is a workaround. Java needs to be backward compatible and as these 'forbidden' classes once were accessible without restraint, you can still get access to them by adding `add-exports` flags to your `java` and `javac` commands. Modules in the JDK have the module-info.java file in their root in which you tell Java, among others, which packages can be used by what other module(s).  

The `--add-exports` is described [here](https://docs.oracle.com/en/java/javase/11/migrate/index.html#GUID-2F61F3A9-0979-46A4-8B49-325BA0EE8B66) and [here](https://dev.java/learn/modules/add-exports-opens/) and [here](https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-2F61F3A9-0979-46A4-8B49-325BA0EE8B66). There is also an `--add-opens` flag, this one is used for reflection purposes.

## Intellij, Maven, command line

It is important to note that the flag is required for both running your program (the `java` command) and for compiling (the `javac` command). 

### Obscure behavior in Intellij

Misunderstandings can occur when working in Intellij, as Intellij has its own ways to deal with this problem. When you introduce forbidden imports in your code Intellij first doesn't seem to understand and then asks you whether you want the appropriate flags added. If you confirm, these flags will be added somewhere you cannot really see them. Now you can compile the code and you can run tests, but running the program still won't work.

### Maven is more transparent

What I did to be able to compile, test and run the program without errors, was to create a Maven setup. It required two things:

- A customized Run/Debug configuration
- An extensive pom.xml file (more later)

### Run/Debug configuration in Intellij for Maven setup

The Run/Debug configuration is to be set in the window that pops up when clicking 'Edit configurations'. Click the + sign to add a configuration and select 'Maven' as its type. Then type `exec:exec` in the field where you can 'insert Maven commands'. Once you have this set, you can run the program with this configuration using the play button, but not before you have compiled the code using the Maven compiler in the Maven menu on the right hand side of the screen.

### The pom.xml

None of this Maven stuff works without a good pom.xml file. You basically need separate Maven build plugins for compiling, running, unit tests and integration tests. This is the list of build plugins to include:

- maven-compiler-plugin (compilation)
- exec-maven-plugin (running)
- maven-surefire-plugin (unit tests)
- maven-failsafe-plugin (integration tests)

Each of these plugin declarations needs to have the --add-exports arguments in their code. At the end of this post the full pom.xml is shown.

### Running via the Powershellcommand line

If I run the program via the command line I use a `mvn` command. The command `mvn exec:exec` works if you want to run stuff, of course the right pom.xml file must be in the root.

Working with the regular `java` and `javac` commands should work as well, but then you have to type the --add-exports flags.

## The pom.xml file that worked for me 

This one did the trick. Note that it has the shade plugin included as well, but that one has nothing to do with --add-exports. Note also that the flags are written down as a general property and then refered to by the different plugin declarations.

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.geertjankuip</groupId>
    <artifactId>ASTVisitor</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <!-- Java -->
        <maven.compiler.release>21</maven.compiler.release>  // This is the right way to declare the version
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!-- Versions -->
        <surefire.version>3.5.4</surefire.version>

        <!-- JVM module flags -->
        <jdk.module.args>
            --add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED
            --add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED
            --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
        </jdk.module.args>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>4.0.0-M1</version>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>5.20.0</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>

            <!-- Unit tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${surefire.version}</version>
                <configuration>
                    <argLine>${jdk.module.args}</argLine>
                </configuration>
            </plugin>

            <!-- Integration tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${surefire.version}</version>
                <configuration>
                    <argLine>${jdk.module.args}</argLine>
                </configuration>
            </plugin>

            <!-- Compile the app -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <release>21</release>
                    <compilerArgs>
                        <arg>--add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED</arg>
                        <arg>--add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED</arg>
                        <arg>--add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED</arg>
                    </compilerArgs>
                </configuration>
            </plugin>

            <!-- Run the app -->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.3.0</version>

                <executions>
                    <execution>
                        <id>run</id>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                    </execution>
                </executions>

                <configuration>
                    <executable>${java.home}/bin/java</executable>

                    <arguments>
                        <!-- JPMS flags -->
                        <argument>--add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED</argument>
                        <argument>--add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED</argument>

                        <!-- classpath -->
                        <argument>-cp</argument>
                        <classpath/>

                        <!-- main class -->
                        <argument>com.geertjankuip.astvisitor.Main</argument>
                    </arguments>
                </configuration>
            </plugin>

            <!-- Fat jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.6.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <minimizeJar>false</minimizeJar>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.geertjankuip.astvisitor.Main</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>

</project>
```


