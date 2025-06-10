# Maven modules and JPMS modules

While trying to figure out how the mave-core module worked I stumbled upon something that seemed illogical to me. I had download the Maven repository from GitHub and opened the whole thing in Intellij. I went deep into the directory tree to arrive at this file:

```
..\maven\impl\maven-core\src\main\java\org\apache\maven\artifact\factory\DefaultArtifactFactory.java
```

I noticed that the return type of many methods was 'Artifact,' and given the importance of 'Artifacts' in Maven I wanted to see its code. The imports showed this ```import org.apache.maven.artifact.Artifact;``` so I started looking for a directory tree resembling ```org.apache.maven.artifact```which was nowhere to find. Using Intellij's Ctrl+click I found out the Artifact.java files resided here:

```
..\maven\compat\maven-artifact\src\main\java\org\apache\maven\artifact\Artifact.java
```

Notice the second position of the path, 'compat', clearly different from 'impl'. I wondered how Intellij was able to know that it should search in a completely different branch and learned something from ChatGPT who explained it.

### Maven modules vs JPMS modules

Maven modules are different from the ones in the Java Platform Module System as I learned them for Oracle SE 11. These are the differences:

- JPMS modules need module-info.java files, Maven modules need module declarations in pom.xml.
- JPMS lets you decide what to shield and what to export, Maven modules do not have this encapsulation principle.
- JPMS needs the module path, Maven works with classpath
- Maven modules can be nested, JPMS modules can't.

Modules in Maven predate those in base Java. They are used as a way to organize code, not more. The Maven code itself is based on this organizational modular system. When you declare modules in the high level pom.xml, you can add dependencies in lower level pom's. 

### Maven's internal module structure

Maven has the following module structure:

```
  <!--maven/pom.xml (level 0)-->
  <modules>
    <module>api</module>
    <module>impl</module>
    <module>compat</module>
    <module>apache-maven</module>
  </modules>

  <!--maven/impl/pom.xml (nested level 1)-->
  <modules>
    <module>maven-support</module>
    <module>maven-impl</module>
    <module>maven-di</module>
    <module>maven-xml</module>
    <module>maven-jline</module>
    <module>maven-logging</module>
    <module>maven-core</module>
    <module>maven-cli</module>
    <module>maven-testing</module>
    <module>maven-executor</module>
  </modules>

  <!--maven/impl/maven-core (nested level 2)-->    
    <dependency>
      <!-- this is a dependency from the level 1 compat module-->
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-artifact</artifactId>
    </dependency>
```

As you see, there are modules in modules, and once there is no module beneath a module anymore, it starts to declare dependencies upon everything it needs from whatever module within the maven module tree. 

### There is also Intellij module

Besides the Maven and JPMS modules there is also the Intellij module, which is of its own type. If Intellij can spot module declarations in the pom or in module-info.java it will stick to the Maven or Java way, but if not it has its own module system, which btw also predates the JPMS. 




