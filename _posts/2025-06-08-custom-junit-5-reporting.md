# Custom JUnit 5 reporting

As the Maven Surefire plugin requires a test library to do the actual testing, I dug into JUnit 5 and was positively surprised. Previously I thought that unit testing was the most boring thing on earth but when you make it part of the Maven build cycle it makes a lot more sense. It seems useful once you have been trying to improve existing code and want to know if you have accidently broken things. 

What annoys me is that the reporting from JUnit is so crap when you work with Maven from the command line. JUnit has annotations that help to create readable reports on test results, namely @DisplayName and @DisplayGeneration but these don't show up in Surefire output, only in IDE's. ChatGPT suggested using Allure as a way to get good testreports but that is sort of too big a solution for now. Btw I tried it a bit but didn't get the configuration right. 

### Hooking into JUnit lifecycle events

There are three ways to add extra code, or to 'extend the code', to the so-called [lifecycle events]() of JUnit:

- Declarative extension registration (@ExtendWith)
- Programmatic extension registration (@RegisterExtension)
- Automatic extension registration

### Automatic Extension Registration

It is the latter of the three I am interested in. It is the global variant of extension registration, meaning that it will apply to all classes. It uses Java's ServiceLoader mechanism. To create an automatic extension you need to take the following steps:

- Create a class that extends either Extension or TestExecutionListener. I put it in the test part of the directory tree. Don't forget a package name. 
- Write its FQN in a file named either org.junit.jupiter.api.Extension or org.junit.platform.launcher.TestExecutionListener. This file must be in src/test/resources/META-INF/services.
- Set the junit.jupiter.extensions.autodetection.enabled configuration parameter to true.
- Make sure to include the right dependencies in the pom.

Important: the files must have a name that exactly matches the FQN _of the interface that you implement_. Only by doing it this way the ServiceLoader will be able to find it. Thus, if it is a TestExecutionListener that you implement, use org.junit.platform.launcher.TestExecutionListener as filename. If it is the Extension interface that you implement, use org.junit.jupiter.api.Extension as filename.

To set the junit.jupiter.extensions.autodetection.enabled configuration parameter to true you add the following code to the Maven Surefire plugin:

```
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.1.2</version>
      <configuration>
        <properties>
          <configurationParameters>
            junit.jupiter.extensions.autodetection.enabled = true
          </configurationParameters>
        </properties>
      </configuration>			  
    </plugin>
```
#### Dependencies

The dependencies you need are the following:

```
  <!-- This one is required for normal JUnit testing, using the @Test, @DisplayName, @ParameterizedTest etc -->
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>	
    <scope>test</scope>
  </dependency>	

  <!-- JUnit Platform Launcher (provides TestExecutionListener, Launcher, etc.) -->
  <dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <scope>test</scope>
  </dependency>

  <!-- Optional: JUnit Platform Engine (required if you want to run tests via Maven Surefire or other tools) -->
  <dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-engine</artifactId>
    <scope>test</scope>
  </dependency>
```

#### TestExecutionListener

[TestExecutionListener](https://junit.org/junit5/docs/current/api/org.junit.platform.launcher/org/junit/platform/launcher/TestExecutionListener.html) is an interface that you can implement to create a class that will be the extension to the code related to the lifecycle events. All eight methods in it are empty default methods, which means that you can choose which ones to override and which ones not to use. These methods get executed on specific moments in the lifecycle of the test and have the following names:

- **dynamicTestRegistered**(**TestIdentifier** testIdentifier)
- **executionFinished**(**TestIdentifier** testIdentifier, **TestExecutionResult** testExecutionResult)
- **executionSkipped**(**TestIdentifier** testIdentifier, **String** reason)
- **executionStarted**(**TestIdentifier** testIdentifier)
- **fileEntryPublished**(**TestIdentifier** testIdentifier, **FileEntry** file)
- **reportingEntryPublished**(**TestIdentifier** testIdentifier, **ReportEntry** entry)
- **testPlanExecutionFinished**(**TestPlan** testPlan)
- **testPlanExecutionStarted**(**TestPlan** testPlan)

**TestIdentifier** is a class containing methods to retrieve information about the test method. ```getDisplayName()``` returns the value of @DisplayName, I was actually looking for this one. ```getTags()``` returns the set of tags for the tested class(container) or method. ```getSource()``` helps to retrieve the name of the class and eventually method that is being tested. Small code example illustrating the latter:

```
Optional<TestSource> source = testIdentifier.getSource();
source.ifPresent(testSource -> {
    if (testSource instanceof MethodSource methodSource) {
        System.out.println("Method: " + methodSource.getMethodName());
        System.out.println("Class: " + methodSource.getClassName());
    } else if (testSource instanceof ClassSource classSource) {
        System.out.println("Class: " + classSource.getClassName());
    }
});
```






 



