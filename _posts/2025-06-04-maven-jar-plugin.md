# Maven jar plugin

The Maven jar plugin can be configured in pom.xml to customize the packaged product. This configuring takes mostly place in the plugins section of the build section. Documentation can be found [here](https://maven.apache.org/plugins/maven-jar-plugin/).

A key line from the documentation: _"The plugin uses Maven Archiver to handle jar content and manifest configuration."_ Maven Archiver belongs to the shared components, which can be utilized by plugins. Its documentation is [here](https://maven.apache.org/shared/maven-archiver/).

For the record, the jar plugin has two goals:

- jar:jar create a jar file for your project classes including resources
- jar:test-jar create a jar file for your project test classes


### Maven Archiver and the manifest file

By default, the jar plugin creates a minimal manifest file with only this:

```
Manifest-Version: 1.0
```

The example below shows how to add entries to a manifest file. Within ```<configuration>``` the archive tag is placed. Setting ```<addClasspath>``` to true results in the creation of a classPath entry in the manifest file. To get the highly prized Main-Class entry, add the ```<mainClass>``` tag. There are many more tags that let you add all sorts of known entries, and if you want custom entries you can use the ```<manifestEntries>``` tag. If you made your own manifest file and want to use it, you can add a tag ```<manifestFile>``` with the file path right below ```<archive>```.

```
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.4.2</version>
        <configuration>
          <archive>
            <manifest>
              <addClasspath>true</addClasspath>
	      <mainClass>com.example.Main</mainClass>  <!-- creating a jar that can be run -->
            </manifest>
            <manifestEntries>
              <mode>development</mode>
              <url>${project.url}</url>
              <key>value</key>
            </manifestEntries>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

Besides manipulating the manifest file, Maven Archiver can do some more on the contents of the packaged file. Compression can be turned of, there is a setting that lets you decide if elements of a previous jar- or other file can be reused when you redo the packaging, and you can decide to leave out the pom.xml and the pom.properties file. All is described in the [documentation](https://maven.apache.org/shared/maven-archiver/).

### Include and exclude content

You might want to add extra content to the jar/war/ etc above the default or omit content. You do this with the ```<include>``` and ```<exclude>``` tags within the configuration section:

```
<build>
  <plugins>
    <plugin>
      <artifactId>maven-jar-plugin</artifactId>
      <version>3.3.0</version>
      <configuration>
        <includes>
          <include>com/example/**</include>
        </includes>
        <excludes>
          <exclude>config/**</exclude>
        </excludes>
      </configuration>
    </plugin>
  </plugins>
</build>
```

The documentation says the following: _"Note that the patterns need to be relative to the path specified for the plugin's classesDirectory parameter."_

The classesDirectory has the reference ${project.build.outputDirectory}. By default it points to target/classes, thus it is the directory were the classfiles are stored. There is logic in it, class files are the major thing stored in the endproduct. The code above includes everything that is in target/classes/com/example/, and excludes everything in target/classes/config/. 

Note: ** means everything that can be recursively found, * means everything directly in that folder but not the nested files and folders. It goes just one level deep.

You can do this as well: ```<include>**/service/*</include>```. This includes everything one level deep in the map service, who's path is target/service/.

#### Includes is a whitelist

The <includes> section acts as a whitelist and will be processed before ```<excludes>```, even if if <excludes> is declared first. You can specify a selection with ```<includes>``` and then remove something from this selection using <excludes>. The process is called file filtering, which is a common pattern in Maven.

### Multiple products

The documentation gives pom code that results in one extra jar file, that has the same name as the regular one but with a classifier/extension of '-client'. The additional jar file has only the folder target/service/, one level deep, as content. 

According to the documentation, "_The jar-plugin must be defined in a new execution, otherwise it will replace the default use of the jar-plugin instead of adding a second artifact. The classifier is also required to create more than one artifact._"

Btw just removing the execution and executions tags won't get you a file with '-client' as suffix and other contents in the jar. This is because ```<classifier>``` and ```<includes>``` do not work outside of the execution context. At least that is what ChatGPT tells.

```
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.4.2</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>jar</goal>
            </goals>
            <configuration>
              <classifier>client</classifier>
              <includes>
                <include>**/service/*</include>
              </includes>
            </configuration>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
```

#### Disabling the default jar

If you want only one file with a '-client' suffix and adjusted content, you need to disable the default jar via execution and use a second execution section for the customized variant you want. The disabling works as follows:

```
        <!-- This disables the default JAR creation -->
        <execution>
          <id>default-jar</id> <!-- 'default-jar' is the internal execution id Maven uses for default JAR generation -->
          <phase>none</phase>
        </execution>

        <!-- This creates the custom JAR -->
        <execution>
          <id>client-jar</id>
          <phase>package</phase>
          <goals>
            <goal>jar</goal>
          </goals>
          <configuration>
            <classifier>client</classifier>
            <includes>
              <include>**/service/*</include>
            </includes>
          </configuration>
        </execution>
```

Of course this is quite a useless operation. To get a jar with other content you can just leave the executions stuff and use includes and excludes. If you want to have another than the default name you can use the tag ```<finalName>```, a direct child of ```<build>```.


### Test jar

Maven documentation distinguishes two ways in which you can save your test classes for reuse in a jar file:

- Create an attached jar with the test-classes from the current project and loose its transitive test-scoped dependencies.
- Create a separate project with the test-classes.

The benefit of the second option is that all the dependencies and the transitive dependencies of the project will be part of the pom.xml of the jar file. In the first option, the dependencies related to testing have scope 'test' and thus will not be included in the generated jar file. The dependency declarations relevant for testing are gone. 

#### Create attached jar with test classes

This is the sample code from documentation. It uses an execution section to add an extra test-jar file. By default this extra jar has a suffix/classifier '-tests': 

```
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.4.2</version>
        <executions>
          <execution>
            <goals>
              <goal>test-jar</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

To use this test-jar in another project (you need to figure out which dependencies to add to the pom of that project to make it work) you declare the test-jar file as a dependency. Note the addition of ```<classifier>``` with a value of 'tests'.

```
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>groupId</groupId>
      <artifactId>artifactId</artifactId>
      <classifier>tests</classifier>
      <type>test-jar</type>
      <version>version</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ...
</project>
```

#### Create a separate project of your test

"_In order to let Maven resolve all test-scoped transitive dependencies you should create a separate project._" This is what the documentation says and the makers of Maven consider this the preferred way.

- _Move the sources files from src/test/java you want to share from the original project to the src/main/java of this project. The same type of movement counts for the resources as well of course._
- _Move the required test-scoped dependencies from the original project to this project and remove the scope (i.e. changing it to the compile-scope). And yes, that means that the junit dependency (or any other testing framework dependency) gets the default scope too. You'll probably need to add some project specific dependencies as well to let it all compile again._

This is the base pom of the new project (note the 'tests' suffix). Further on in this pom you'll find all the required dependencies, copied from the base project, with scopes converted from 'test' to 'compile':

```
<project>
   <groupId>groupId</groupId>
    <artifactId>artifactId-tests</artifactId>
    <version>version</version>
  ...
</project>
```

Add this new test project to the project where you need it:

```
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>groupId</groupId>
      <artifactId>artifactId-tests</artifactId>
      <version>version</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ...
</project>
``` 

#### Using maven-dependency-plugin to make this work

I'm not completely satisfied with this solution, because upon running ```mvn test``` in this scenario Maven will not run the tests you just added as a dependency. I asked ChatGPT and he confirmed this. ChatGPT gave a solution as well, namely configuring the pom.xml file with the Dependency plugin so that this plugin will unpack the test-jar and put the files in ${project.build.testOutputDirectory} (ie target/test-classes/). This is the code:

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>3.6.1</version>
  <executions>
    <execution>
      <id>unpack-shared-tests</id>
      <phase>process-test-classes</phase>
      <goals>
        <goal>unpack</goal>
      </goals>
      <configuration>
        <artifactItems>
          <artifactItem>
            <groupId>com.example</groupId>
            <artifactId>shared-tests</artifactId>
            <classifier>tests</classifier>
            <type>test-jar</type>
            <outputDirectory>${project.build.testOutputDirectory}</outputDirectory>
          </artifactItem>
        </artifactItems>
      </configuration>
    </execution>
  </executions>
</plugin>
```

This should work fine. It is something new for me, it is interesting to see the phase ('process-test-classes') and the goal ('unpack'). I might study this maven-dependency-plugin a bit more.

This is it for now.





