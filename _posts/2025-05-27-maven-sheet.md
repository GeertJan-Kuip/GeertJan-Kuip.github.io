# Maven sheet

Below a lot of annotated .xml file with all of the relevant tags that can be used in Maven's pom.xml. To be used as a reference. It is a work in progress. I used Maven documentation and ChatGPT as guide. [This page](https://maven.apache.org/pom.html) gives much explanation and I used it a lot. 

Nice example for a pom.xml in action: [Maven's base pom](https://github.com/apache/maven/blob/master/pom.xml)


### Minimal POM

A minimal pom can do with very little. Use modelVersion (use 4.0.0). The properties I added here can be omitted but it helps to add them, Maven output might complain if you omit it. ChatGPT strongly advised me to enforce UTF-8 for consistency across platforms. Btw the 'maven.compiler.release' tag is a wondeful thing.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>

	<groupId>
	<artifactId>
	<version>

	<properties>
		 <java.version>21</java.version>
		 <maven.compiler.release>${java.version}</maven.compiler.release>		
		 <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		 <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		 <project.build.outputEncoding>UTF-8</project.build.outputEncoding> 
	</properties>
```

### Packaging

'Packaging' is not required because it has a default of jar that is often okay. The interesting value is 'pom' that is used either for parent projects (you can even make a minimal project consisting of a folder with artifactId name and a pom.xml) in which you resolve dependency versions in the dependencyManagement section, or for multi-module projects where the are stored in the root folder.

```
	<packaging> jar (default), war, rar, pom, ear, maven-plugin, ejb.
```

### Dependencies

Dependencies is where it began with Maven. groupId and artifactId mandatory. Version is mandatory unless declared in dependencyManagement or parent POM. The basics:

```
	<dependencies>
		<dependency>
			<groupId>
			<artifactId>
			<version> <!-- mandatory, unless declared in parent pom or <dependencyManagement> -->
			<type> <!-- jar(default),war,ejb,ejb-client,test-jar, java-source,javadoc,bundle -->
			<classifier> <!-- postfix to version, like -tests, -sources or -javadoc  -->
```

#### Scope

Scope is an important one and can have the values compile(default), provided, runtime, test or system.

```         
			<scope>
```

- **compile** (default). Dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.
- **provided**. At runtime you expect the JDK to provide the dependency.
- **runtime**. Maven includes a dependency with this scope in the runtime and test classpaths, but not the compile classpath.
- **test**. Dependency not required for normal use, only available for the test compilation and execution phases. Not transitive.
- **system**. See below. Requires <systempath> to be set.

#### Systempath

```
			<systemPath>
```

Is used only if the dependency scope is system. The path must be absolute, so it is recommended to use a property to specify the machine-specific path. When scope is system, the depencency will not be searched for in repositories but only on the path provided here. You must ensure the dependency is available on the path on every machine in every phase. Note: this feature is deprecated.

#### Optional

```

			<optional> <!-- default is false -->
```

If ```<optional>``` is set to true, the dependency is included but if this project becomes a dependency for some other project, it is not included anymore. Prevents including too many unused transitive dependencies (think of all possible database drivers).

#### Exclusions

```
			<exclusions>
				<exclusion>
					<groupId>org.apache.maven</groupId>
					<artifactId>maven-core</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>
```

Under exclusions you list transitive dependencies of the dependency at hand that you want to exclude for some reason. It might still be possible that this excluded dependency enters your project via another route, and that is okay and that might even be your aim. Maven itself calls exclusions an option of 'last resort.' You can use wildcards * in groupId and artifactId to exclude all transitive dependencies.

### Parent

The parent's packaging value must be 'pom' if you declare a parent. All values of parent pom are inherited, except for artifactId, name, prerequisites and profiles. There is a super POM from which every pom inherits. To see how the super pom affects your pom, make your own pom and use 'mvn help:effective-pom'. It will show even things that are not in the super POM but do their work by default anyway. 

An effective use of parent pom is centralized uniform version resolution. Create a dependencyManagement section in the parent and declare versions of dependencies. In the child poms you still have to declare the dependency but you can omit version number. Always check the dependency tree to avoid unwanted effects. ```<relativePath>``` is optional. Tells where to search for the parent pom. If not provided, will search in the local and remote repositories.

```			
	<parent>
		<groupId>org.codehaus.mojo</groupId>
		<artifactId>my-parent</artifactId>
		<version>2.0</version>
		<relativePath>../my-parent</relativePath>  <!-- optional -->
	</parent>
```

### Modules

For modular builds to work, packaging of parent pom must be set to 'pom'. Poms of modules do not have to declare a parent. It is called 'aggregation'. It is possible and might be good practice to combine 'inheritance' with 'aggregation.' In ```<modules>``` you list the names of the module base directories, or the path of the pom.xml files that reside in those directories. Module order is not important here, Maven figures it out itself based on the module poms. If a module requires another module, it will have to declare that other module as a dependency and that gives Maven the possibility to create a dependency tree.

```
	<modules>
		<module>my-project</module>
		<module>another-project</module>
		<module>third-project/pom-example.xml</module>	
	</modules>
```

#### Modules, file placement and naming

I was confused about naming conventions for modules. In Java world you use dot.case, in Maven world kebab-case (with hyphens). ChatGPT explained these two can live together. For Maven, name the module folders kebab-case and refer to them in your poms with that name. For module-info.java, use dot.case, yes, for the same module. For every module, the pom.xml must be in the root, while the module-info.jave file must ly deeper, in the Java folder, just beneath where the groupId folder names start. In general, every module has an extensive directory tree as to follow Maven conventions. Summary:

```
my-project/my-module/pom.xml
my-project/my-module/src/main/java/module-info.txt
my-project/my-module/src/main/java/com/geertjankuip/somename/<here are the packages, java files>
```

### Dependency management			


Here you resolves version conflicts of transitive dependencies. Without dependencyManagement, Maven picks the nearest version of a transitive dependency to your project or the first it encounters. With dependencyManagement you can decide which version to use of a transitive dependency. 

#### Minimum and maximum versions

There is special syntax that gives precise control over allowed versions. 

Difference between soft and hard requirement and using of either [] or (). Soft requirement: version written without brackets or parentheses. 'Use 1.0 if no other version appears earlier in the dependency tree.' Hard requirement: use brackets/parentheses. 

- [1.0] : use only 1.0 ; 
- [1.0,) : use 1.0 or higher ; (,1.0] use 1.0 or lower ; 
- [1.0,2.0) : any version between 1.0 and 2.0 but not 2.0 itself; 
- (,1.2),(1.2) : any version except 1.2 ; 
- (,1.2],[1.4,) : (1.2 or lower) or (1.4 or higher)

Maven has its own rules for reading version strings and comparing different versions (which one is greater). Too much detail to describe here.

Transitive dependencies can be excluded, just like in the ```<dependencies>``` section.

```			
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>
				<artifactId>
				<version>
				<exclusions>
					<exclusion>
						<groupId>com.group.something</groupId>
						<artifactId>excluded-artifact</artifactId>
					</exclusion>
```

#### Type and scope

The ```<type>``` tag, child of ```<dependency>```, has jar as default and the options differ from those of dependency declarations in ```<dependencies>```. Note that 'pom' can be a type here, but not in a dependency that is not part of the dependencyManagement section.

There is a child tag ```<scope>``` equal to that of regular dependency but with one extra scope type, namely 'import'. Import requires 'pom' as dependency type. Projects can import managed dependencies from other projects. This is accomplished by declaring a POM artifact as a dependency with a scope of "import". The typical practice is to create a 'BOM' (Bill of Materials). This is a parent pom.xml with packaging value of 'pom'. It has a dependencyManagement section where you specify the versions of libraries, and it can be used by different projects to inherit from. The BOM has the form of the simplest java project, only a root folder (named after artifactID) with a pom.xml file in it. The BOM can be put into a central repository and used by everyone for consistency. When using import, beware of circular dependencies and never do import a pom that is in a submodule.

```

				<type> <!-- jar (default),war,ejb,ejb-client,test-jar,java-source,javadoc,bundle,pom,bar -->
					
				<scope> <!-- compile (default), provided, runtime, test, system, import -->
```

### Properties

Maven properties can be called by value placeholders (${varName}), anywhere in the POM. You can refer to specific variables:
- ${env.X} refers to environment variable X, as in ${env.PATH}. The names of environment variables are normalized to all upper-case.
- ${project.X} refers to variable in pom, as in ${project.version}
- ${settings.X} refers to variable in settings.xml, as in ${settings.offline}. It doesn't work the other way around, as settings.xml is loaded before pom.xml.
- ${java.X} refers to Java system variable, such as ${java.home}.
- ${X} refers to a variable set under the properties tag, such as ${someVar}.

The sample below shows 2 self-made properties and one property with a special meaning, the name is defined.

```
	<properties>
		<someVar>value</someVar>
		<anotherVar>value</anotherVar>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
```

### Build		

Build does some special thing that relates to inheritance. Build can be a top-level section on its own but it can also be a section under 'profile' or under 'plugin'. A build section can also be in a parent pom and be inherited. 

#### Build and BaseBuild

Maven distinguishes between the 'BaseBuild' type and the 'Build' type. Elements belonging to the BaseBuild type can be used in a top-level build section and in a nested build section (under profiles). Elements belonging to the Build type can only be part of a top-level build section. Practically this means that <extensions> and <..Directory> cannot be used in a build section in a profile as they belong to the Build type. 

Some confusion exists about ```<plugins>```. https://maven.apache.org/xsd/maven-4.0.0.xsd implicates that ```<plugins>``` cannot be part of ```<profile><build>```, but the Maven documentation states that they can. ChatGPT struggled with it and suggested that even though the .xsd is very strict, Maven allows a ```<plugin>``` section in ```<profile><build>```. So it is just ```<..Directory>``` (there are several) and ```<extensions>``` that cannot be used in ```<profile><build>```.	
	
```	
	<build>	
		<defaultGoal>install</defaultGoal> <!-- If no goal is provided somehow, Maven resorts to this value. It means you can just type 'mvn' in the command line.-->
		<directory> <!-- Default: ${project.basedir}/target. This is where the resulting file(s) is/are saved.-->
		<finalName> <!-- Default: ${artifactId}-${version}. This is the name of the resulting file. It is possible that the name will be extended with a classifier defined elsewhere.-->
```

#### Filters

The default value for ```<filters>``` is ${project.basedir}/src/main/filters/. The file(s) you put here contain key-value pairs that will be used to replace the appropriate placeholders in configuration files, following the principles of resource filtering. Resource filtering let you adjust configuration files, so that for example the version mentioned in the pom or in a filter will also be part of application.properties. Perfect for packaged versions that need actual and relevant info in their metadata/config files.

Filter files have a .properties extension and their default location is ${project.basedir}/src/main/filters/. Nevertheless, you must specify them individually. No wildcards possible. What you can do is put a property-placeholder here (${env.filter}) and supply the filter path as -Denv.filter=PathToFile argument in the build command.

```
		<filters>			
			<filter>  </filter>				
```

#### Resources
		
Resources indicate where the resources are. Default is src/main/resources. You can omit the resources section but not if you want to set 'filtering' to true for the src/main/resources or any other folder, because setting it to true must be done within ```<resource>``` With filtering, placeholders in configuration files found here will be replaced with their counterparts in pom, env, system and filter files. The resources section is already available in the super POM but without the filtering tag. As the default value of filtering is false, filtering is not enabled by default.

```
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
			</resource>
```

#### More extended resource example
		
This is code from the Maven documentation. I asked ChatGPT why ```<targetPath>``` and ```<directory>``` where split the way they were, it looked arbitrarry. Got a good explanation: upon compiling, the configuration.xml file will be stored in target/classes/META-INF/plexus. The ```<targetPath>``` directory tree is copied, the ```<directory>``` directory tree isn't. In other words, targetPath lets you control the directory layout within the classes directory and thus in the final jar.

```
		<resources>	
			<resource>
				<targetPath>META-INF/plexus</targetPath>
				<filtering>false</filtering>
				<directory>${project.basedir}/src/main/plexus</directory> <!-- Which files to include. Wildcards allowed. -->
				
				<includes>
					<include>configuration.xml</include>
				</includes>				
				<excludes>
					<exclude>**/*.properties</exclude> <!--Which files to exclude. In conflicts between include and exclude, exclude wins.-->
				</excludes>

			</resource>
```

### Testresources
			
What you can do within ```<resources>``` can be done within ```<testResources>``` as well. Testresources are not deployed. The default directory is a different one so I mention it below.

```
		<testResources>
			<testResource>
				<!-- The default directory is ${project.basedir}/src/test/resources. You need to specify if you want another one.-->
				<directory>${project.basedir}/src/myTest/utilityfiles</directory>
				...
			</testResource>
```

### Plugins
			
Plugins are very relevant in customizing builds. You often need to use the ```<executions>``` section to tell Maven what the plugin should do (goal) and when (phase). This below is code copied from the Maven documentation with my annotations.

```
		<plugins>
			<plugin>				
				<groupId>org.apache.maven.plugins</groupId> <!--For groupId 'org.apache.maven.plugins' is the default so here it could be omitted.-->					
				<artifactId>maven-jar-plugin</artifactId>
				<version>2.6</version>	
```

#### Extensions

The extensions tag can be set to true or false, default is false. Some plugins need this set to true to be able to do something. Under the hood it means that such plugins start to mess with Maven internals, possibly with side effects. Therefore it needs a specific declaration, or you might say permission, to do this.

```			
				<extensions>false</extensions> <!-- Default value for extensions is 'false'. -->
```

#### Inherited

Default is true. If true, child poms will inherit the plugin configuration as defined here.

```									
				<inherited>true</inherited> 
```

#### Configuration
					
				
The specific values to set within the configuration and their defaults are plugin-specific. You need to look them up via help or documentation. If your POM declares a parent, it inherits plugin configuration from either the build/plugins or pluginManagement sections of the parent. The rules for plugin configuration inheritance are very specific. It defaults to merge but this can be adjusted with combine.children and combine.self. 

```
				<configuration>
					<classifier>test</classifier>
				</configuration>
```				
				
You can have nested stuff. Below a code sample where the configuration gets more extense. combine.children and combine.self are added to regulate inheritance behavior. Again, documentation explains it better. (https://maven.apache.org/pom.html#advanced%20configuration%20inheritance)

```
				<configuration>
					<items combine.children="append">
						<item>parent-1</item>
						<item>parent-2</item>
					</items>
					<properties combine.self="override">
						<parentKey>parent</parentKey>
					</properties>
				</configuration>
```

#### Dependencies (plugin)
				
You can be specific about the dependencies of the plugin. This nested dependencies section works like the top level dependencies section. Two main uses for adding this dependencies here is to change a version of a dependency or to exclude a transitive dependency. 

```
				<dependencies>...</dependencies>
					<dependency>
						<groupId>
						<artifactId>
						<version>
						<type>
						<classifier>
						<scope>
						<systemPath>
						<optional>
						<exclusions>
							<exclusion>
								<groupId>org.apache.maven</groupId>
								<artifactId>..</artifactId>
							</exclusion>			
```

#### Execution

The plugins 'maven-antrun-plugin' and 'maven-assembly-plugins' are both plugins that can be active in different phases with different goals. To specify what they should exactly do you use this <executions> section. Below is a possible executions section for the antrun plugin. Note: you can add multiple <execution> sections within <executions>, for example if you want the assembly plugin to generate multiple files on one Maven run.

```
				<executions>
					<execution>
						<id>echodir</id> <!--Setting an id value is optional but it makes Maven output better readable (it uses the id values in there).-->							
						<goals>
							<goal>run</goal> <!-- You can specify more than one goal -->								
						</goals>
						<phase>verify</phase> <!-- ChatGPT listed 23 different phases. Quite a lot.. -->							
						<inherited>false</inherited> <!-- Only relevant for parent poms. Children will inherit execution values. -->							
						<configuration>
							<tasks>
								<echo>Build Dir: ${project.build.directory}</echo>
							</tasks>
						</configuration>
					</execution>
```

Below another example, I wanted to use the shade plugin in a aggregated (multi-module) project to create a single, runnable jar of all the modules. By default Maven will create a separate jar for every module. In the pom of the central module I had to put this to make it work:

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
							<createDependencyReducedPom>false</createDependencyReducedPom>
							<transformers>
								<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
									<mainClass>com.geertjankuip.garage.Garage</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
```

### Pluginmanagement

		
Just like there is <dependencyManagement> there is also <pluginManagement>. What you define here is not for the project itself but for every POM that inherits from this POM. The child must declare this project as parent to make it work. This makes it different from inheritance of the values of the dependencyManagement section, that can be inherited without a parent-child relationship (via import). This here is from the documentation as example. A child inheriting does only have to declare groupId and artifactId to make the plugin work for itself.

```
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-jar-plugin</artifactId>
					<version>2.6</version>
					<executions>
						<execution>
							<id>pre-process-classes</id>
							<phase>compile</phase>
							<goals>
								<goal>jar</goal>
							</goals>
							<configuration>
								<classifier>pre-process</classifier>
							</configuration>
						</execution>
					</executions>
				</plugin>
			</plugins>
```

### Build Type elements
		
#### source- and other directories
	
Now we arrive at the elements that belong to the Build type and can only be used in <project><build>. No override from more nested build sections is possible. First is <..Directory>. There are several types of directories that can be set. If the values of a *Directory element is set as an absolute path (when their properties are expanded) then that directory is used. Otherwise, it is relative to the base build directory: ${project.basedir}.

```		
	<build>
		<sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
		<testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
		<outputDirectory>${project.basedir}/target/classes</outputDirectory>
		<testOutputDirectory>${project.basedir}/target/test-classes</testOutputDirectory>
	</build>
```

#### Extensions		
		
The other Build type element is extensions. Extensions are a bit obscure, as ChatGPT noted. Extensions are different from a plugin as they do not have a specific task in the build process. Instead, they alter the build process itself. They are loaded before the plugins and change the way Maven works. The example below is sort of typical. The wagon-ftp extension enables Maven to use FTP as a transport protocol when interacting with remote repositories. Maven doesn't have this capability by default. It thus alters the specification of the methods Maven uses under the hood during build processes.

```
	<build>
		<extensions>
			<extension>
				<groupId>org.apache.maven.wagon</groupId>
				<artifactId>wagon-ftp</artifactId>
				<version>1.0-alpha-3</version>
			</extension>
		</extensions>
	</build>	
```		

### Reporting

Reporting is a top-level tag that. Just like 'build' configures the built product, 'reporting' configures the generated documentation. Reporting is part of the 'site' phase and can include the generation of static webpages, javadoc content etc. There are specific plugins that can be used here, just as there are specific plugins for build. You can very precisely configure things, whereby the role of <execution> is played by <reportSet>. 
		
Important note: the maven-javadoc-plugin can be found in the super-POM under <profile><build>, not under <reporting>. This is because maven-javadoc-plugin can be used to create a Jar with metadata that is packaged with the rest of the build product during the package phase, not the 'site' phase.	

One more note: the reporting section is only tied to the 'site' phase. Therefore you can say it doesn't have the impact that the <build><plugins> section has.

Another note: 'mvn site' command automatically runs the maven-project-info-reports-plugin, even though this plugin isn't mentioned in the superPOM. You can turn this defalt behavior off with the Boolean excludeDefaults element. A more verbose way is shown below (note the amount of nesting):

		
```
	<reporting>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-project-info-reports-plugin</artifactId>
				<reportSets>
					<reportSet>
						<reports>
						<!-- Empty list, no report tags, turns it off -->
						</reports>
					</reportSet>
				</reportSets>
			</plugin>
		</plugins>
```


Another example with a heavy configuration. The maven-javadoc-plugin has multiple goals available, in this example we only want the javadoc goal (and not one of the other 15 goals). In the configuration there apparently is a 'link' value that can be set, which is done in the configuration setting.

```
	<reporting>
		<plugins>
			<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-javadoc-plugin</artifactId>
				<reportSets>
					<reportSet>
						<id>sunlink</id>
						<reports>
							<report>javadoc</report>
								javadoc is one of the 16 possible goals. See https://maven.apache.org/plugins/maven-javadoc-plugin/
						</reports>
						<inherited>true</inherited>
							Children will inherit this reporting plugin configuration
						<configuration>
							Generally, the <configuration> section is the workhorse, not the <reports> section.
							<links>
								<link>http://java.sun.com/j2se/1.5.0/docs/api/</link>
							</links>
						</configuration>
					</reportSet>
				</reportSets>
			</plugin>
		</plugins>
```

### General project information

``
	<name> <!--Name differs from artifactId. The latter has a somewhat technical name with hyphens. Under name you can be poetic: Word, AutoCAD or 'GitHub Desktop'-->
		
	<description> <!-- A short, human readable description of the project. -->
		
	<url> <!-- The project's home page -->
		
	<inceptionYear> <!-- The year the project was first created. -->
```

### Licenses

Licenses are legal documents defining how and when a project (or parts of a project) may be used. ```<distribution>```describes how the project may be legally distributed. The two stated methods are repo (they may be downloaded from a Maven repository) or manual (they must be manually installed).

```
	<licenses>		
		<license>
			<name>Apache-2.0</name>
			<url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
			<distribution>repo</distribution>				 
			<comments>A business-friendly OSS license</comments>
		</license>
```

### Organization, developers, contributors

Under organization, only two child elements are allowed, name and url. For more, do a workaround via the properties section. 

<developers> and <contributors> have many available fields, the same for each. Note that you can add extra info under the properties section.

```
	<organization>
		
		<name>Codehaus Mojo</name>
		<url>http://mojo.codehaus.org</url>
		
	<developers>
		<developer>
			<id>jdoe</id>
			<name>John Doe</name>
			<email>jdoe@example.com</email>
			<url>http://www.example.com/jdoe</url>
			<organization>ACME</organization>
			<organizationUrl>http://www.example.com</organizationUrl>
			<roles>
				<role>architect</role>
				<role>developer</role>
			</roles>
			<timezone>America/New_York</timezone>
			<properties>
				<picUrl>http://www.example.com/jdoe/pic</picUrl>
			</properties>

	<contributors> <!-- Same fields as <developers> -->		
```

## Environment settings

There are many high level tags concerning environment settings, therefore this big heading.


### Issuemanagement

This section contains info on defect tracking system. Mainly for documentation purposes.

```
	<issueManagement>		
		<issueManagement>
			<system>Bugzilla</system>
			<url>http://127.0.0.1/bugzilla/</url>
		</issueManagement>
```

### ciManagement

Stands for Continuous Integration Management, tools that do automatic builds upon timing or triggers. Maven has captured a few of the recurring settings within the set of notifier elements. Example:

```	
	<ciManagement>		
		<system>continuum</system>
		<url>http://127.0.0.1:8080/continuum</url>
		<notifiers>
			<notifier>
				<type>mail</type>
				<sendOnError>true</sendOnError>
				<sendOnFailure>true</sendOnFailure>
				<sendOnSuccess>false</sendOnSuccess>
				<sendOnWarning>false</sendOnWarning>
				<configuration><address>continuum@127.0.0.1</address></configuration>
			</notifier>
		</notifiers>
```

### Mailinglists

Mailing lists are a great tool for keeping in touch with people about a project. Most mailing lists are for developers and users. Example from docs:

```
	<mailingLists>
		
		<mailingList>
			<name>User List</name>
			<subscribe>user-subscribe@127.0.0.1</subscribe>
			<unsubscribe>user-unsubscribe@127.0.0.1</unsubscribe>
			<post>user@127.0.0.1</post>
			<archive>http://127.0.0.1/user/</archive>
			<otherArchives>
				<otherArchive>http://base.google.com/base/1/127.0.0.1</otherArchive>
			</otherArchives>
		</mailingList>
```

### SCM

Software Configuration Management, also called Source Code/Control Management or, succinctly, version control.

```
	<scm>
		
		<connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
		<developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
		<tag>HEAD</tag>
		<url>http://127.0.0.1/websvn/my-project</url>
```

### Prerequisites

The POM may have certain prerequisites in order to execute correctly. The only element that exists as a prerequisite in POM 4.0.0 is the maven element, which takes a minimum version number.	

```		
	<prerequisites>		
		<maven>2.0.6</maven>
```

### Repositories

Whenever a project has a dependency upon an artifact, Maven will first attempt to use a local copy of the specified artifact. If that artifact does not exist in the local repository, it will then attempt to download from a remote repository. It uses default locations but if you want to modify the defaults, do it here.

The ```<updatePolicy>``` element specifies how often Maven tries to update its local repository from the remote repositories. The choices are: always, daily (default), interval:X (where X is an integer in minutes) or never (only downloads if not yet existing in the local repository).

The ```<checksumPolicy>```tag: When Maven deploys files to the repository, it also deploys corresponding checksum files. Your options are to ignore, fail, or warn on missing or incorrect checksums. The default value is warn.

The ```<id>```tag connects the repository with the servers from settings.xml. Its default value is 'default'. The id is used also in the local repository metadata to store the origin.

```	
	<repositories>

		<repository>
			<releases>
				<enabled>false</enabled> <!-- true or false for whether this repository is enabled for the respective type (releases or snapshots). By default this is true. -->					
			</releases>

			<snapshots>
				<enabled>true</enabled> <!-- default true -->					
				<updatePolicy>always</updatePolicy>					
				<checksumPolicy>fail</checksumPolicy>					
			</snapshots>

			<name>Nexus Snapshots</name> <!-- An optional name for the repository. Used as label when emitting log messages related to this repository. -->
				
			<id>snapshots-repo</id> <!-- default value is 'default' -->
				
			<url>https://oss.sonatype.org/content/repositories/snapshots</url>

			<layout>default</layout> <!-- You probably never use this, has to do with legacy Maven 1. Default value is 'default' -->
				
		</repository>	
```

### Pluginrepositories

The structure of the pluginRepositories element block is similar to the repositories element. They could have merged them into one.

```	
	<pluginRepositories>
```

### Distributionmanagement

This section defines how Maven should distribute/publish the project's output, such as uploading artifacts to a repository, deploying to a website, handling relocation or publishing metadata. It is used by the deploy phase (mvn deploy) and by the site:deploy goal.	

```		
	<distributionManagement>

		<repository> <!-- Specifies the release repository where Maven should upload your final (non-snapshot) artifacts. -->			
			<id>releases</id>
			<url>https://repo.mycompany.com/releases</url>
		</repository>
		
		<snapshotRepository> <!-- Specifies the snapshot repository for deploying snapshot versions (1.0.0-SNAPSHOT). This is kept separate from <repository> because snapshot artifacts have different handling: they are mutable, versioned with timestamps, etc. -->			
			<id>snapshots</id>
			<url>https://repo.mycompany.com/snapshots</url>
		</snapshotRepository>
		
		<site> <!-- Defines where to deploy your project's generated documentation site (mvn site:deploy) -->
			
			<id>my-site</id>
			<url>scp://example.com/www/docs/project</url>
		</site>
		
		<relocation> <!-- Used to deprecate or move a project to a new groupId/artifactId/version. This tells consumers that your artifact has been relocated. -->			
			<groupId>com.newgroup</groupId>
			<artifactId>new-artifact</artifactId>
			<version>2.0.0</version>
			<message>This artifact has moved</message>
		</relocation>
		
		<downloadUrl>https://downloads.example.com/artifacts/myapp-1.0.0.jar</downloadUrl> <!-- Deprecated because nowadays it is managed automatically. Used to specify an external URL where the binary can be downloaded. -->	
			
		<status>released</status> <!-- Used for publishing the stability or maturity level of the project. Valid values are converted, partner, deployed, verified, released. -->
```

### Profiles			

With profiles you can sort of overhaul any pom setting. Key is the <activation> section within <profile>. If you want to know which profile will be activated, use ```mvn help:active-profiles```. Profiles can be activated manually via the -P flag, followed by a comma-delimited list of profile IDs to use. Detailed info [here](https://maven.apache.org/guides/introduction/introduction-to-profiles.html). Here a sample:			

```		
	<profiles>
		<profile>
			<id>test</id>
			<activation>
				<activeByDefault>false</activeByDefault>
				<jdk>21</jdk>
				<os>
					<name>Windows 10</name>
					<family>Windows</family>
					<arch>amd64</arch>
					<version>10.0.19045.5247</version>
				</os>
				<property>
					<name>sparrow-type</name>
					<value>African</value>
				</property>
				<file>
					<exists>/home/jenkins/82467a7c/workspace/aven_maven-box_maven-site_master/file2.properties</exists>
					<missing>/home/jenkins/82467a7c/workspace/aven_maven-box_maven-site_master/file1.properties</missing>
				</file>
			</activation>
			
			// If a profile is activated, you can configure with the following tags:

			<id>test</id>
			<activation>...</activation>
			<build>...</build>
			<modules>...</modules>
			<repositories>...</repositories>
			<pluginRepositories>...</pluginRepositories>
			<dependencies>...</dependencies>
			<reporting>...</reporting>
			<dependencyManagement>...</dependencyManagement>
			<distributionManagement>...</distributionManagement>
		</profile>		
	
</project>
  
```
  

  


