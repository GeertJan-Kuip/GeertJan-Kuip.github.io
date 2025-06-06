# Maven shade plugin

The Maven shade plugin is used for generating fat jars. It has one goal, shade, and is bound to the package phase. When creating a fat jar it gives precise control over what to include or exclude. You need to configure it with an executions section, otherwise it won't do anything. By doing so you get two jars, one regular and one fat. You recognize them because of the name and the filesize. 

The webpage with all specifics is [here](https://maven.apache.org/plugins-archives/maven-shade-plugin-3.6.0/shade-mojo.html).

### Selecting content

Below is an example from the [documentation](https://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.html), or more precisely the execution block declared within ```<plugin><executions>```.

```
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <artifactSet>
                <excludes>
                  <exclude>classworlds:classworlds</exclude>
                  <exclude>junit:junit</exclude>
                  <exclude>jmock:*</exclude>
                  <exclude>*:xml-apis</exclude>
                  <exclude>org.apache.maven:lib:tests</exclude>
                  <exclude>log4j:log4j:jar:</exclude>
                </excludes>
              </artifactSet>
            </configuration>
          </execution>
```

Artifacts are denoted by a composite identifier of the form groupId:artifactId[[:type]:classifier]. I wondered about the ```junit:junit``` identifier, according to ChatGPT JUnit 4 actually had junit as groupId and junit as artifactId. The rules are thus followed here. Other Maven plugins use this composite identifier as well but you should be careful to check the documentation, as sometimes they deal differently with [:type] and [:classifier].

Instead of using ```<excludes><exclude>``` you can use ```<includes><include>``` as well to make a whitelist, and I suppose you can combine both whereby ```<includes><include>``` is processed first.

It is even possible to exclude certain classes within dependencies using filters and eventually wildcards. The syntax is on the documentation page.

### Removing unused classes of dependencies

If you want to minimize the size of the resulting jar you can omit the unused classes from the dependencies. This code with ```<minimizeJar>``` will do that, you can check the [documentation](https://maven.apache.org/plugins-archives/maven-shade-plugin-3.6.0/shade-mojo.html#minimizeJar) of it:

```
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.6.0</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <minimizeJar>true</minimizeJar>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

You can have more fine grained control with filters, includes and excludes, for example if you want to keep some but not all unused classes in your jar file.

### Naming the fat and skinny jar

According to the documentation, the fat jar will replace the default jar as the only file being produced. ChatGPT disagrees and says that by default, if you configure the shade plugin with an executions section, you get two jars, one default skinny and one fat according to your specific configuration. **Update:** I tried and ChatGPT seems to be right, there is always a skinny jar.

The tag ```<shadedArtifactAttached>``` does not prevent the skinny jar to be produced but lets you choose which of the two jars will have the default name. Its default value is false, which means that the fat jar will be named ```artifact-name-2-1.0-SNAPSHOT.jar``` and the skinny jar will have a prefix and be named ```original-artifact-name-2-1.0-SNAPSHOT.jar```. 

If you set ```<shadedArtifactAttached>``` to true, the naming will be different. The skinny jar gets the base filename, while the fat jar will get the suffix 'shaded' (```artifact-name-2-1.0-SNAPSHOT-shaded.jar```). 


```<shadedArtifactId>``` can be used but only works if ```<shadedArtifactAttached>``` is set to true. It replaces the regular name based on artifactId with your value, resulting in a name like ```somerandomnewname-1.0-SNAPSHOT-shaded.jar```. The skinny jar will have a regular name.

```
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.6.0</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <shadedArtifactAttached>true</shadedArtifactAttached> <!-- '-shaded' suffix -->
            <shadedArtifactId>${project.artifactId}</shadedArtifactId>
          </configuration>
        </execution>
      </executions>
    </plugin>
```

Btw with ChatGPT I tried to make a configuration in which only a skinny jar is created but didn't succeed.

### Executable JAR

To get an executable jar with the right manifest you need to use the so called [resource transformer](https://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html). The term "ManifestResourceTransformer" you see below is one of the resource transformers. It enables you to add entries to the manifest file. 

```
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <createDependencyReducedPom>false</createDependencyReducedPom> <!-- default is true -->
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.geertjankuip.garage.Garage</mainClass> 
          </transformer>
        </transformers>
      </configuration>
    </execution>
```

#### createDependencyReducedPom

The ```<createDependencyReducedPom>``` tag is relevant, or at least the 'dependency-reduced-pom.xml' that is generated by default is relevant. The reason the makers have chosen to add this feature is that when others use your shaded jar as a dependency, there will be double loading of dependencies. Maven does not know that the JAR contains the dependency files and will reload them again from the repository. This might result in conflicts. 

To avoid this, you can use the dependency-reduced-pom.xml from which all the dependencies that are bytecode wise part of the jar file are omitted. When someone else pulls your fat jar in as a dependency, he will not load those dependencies that are already contained in the fat jar.

If your fat jar is meant to be used as a library by others, it is good practice to replace its pom.xml with the generated 'dependency-reduced-pom.xml'. You do not have to do anything for that, if ```<createDependencyReducedPom>``` is set to true Maven will create the dependency-reduced-pom.xml and use the content of it to create the pom.xml in the fat jar.

Far better is not to use fat jars as libraries at all. Only when the included dependency files cannot be retrieved from the repositories you should consider packing them in a library jar.

If you created the fat jar just to have something that is runnable, you can set ```<createDependencyReducedPom>``` to false as you won't need dependency-reduced-pom.xml. But if you forget to set this flag, it is no problem, it will work anyway (the required class files are all in the jar, the pom has no influence on the execution of the program).

### Creating your own Shader implementation

Yes, you can create your own variant of this plugin. I'm not gonna do but it is described [here](https://maven.apache.org/plugins/maven-shade-plugin/examples/use-shader-other-impl.html).


