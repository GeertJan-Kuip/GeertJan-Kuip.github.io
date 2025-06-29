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

- Create a class that extends either Extension or TestExecutionListener (there are only 2 you can extend). I put it in the test part of the directory tree. Don't forget a package name. 
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

I copied an implementation of TestExecutionListener that ChatGPT provided and it worked after I did all the configuration things right. Code:

```
package com.geertjankuip.test; // package name terribly chosen, confusing

import org.junit.platform.engine.TestExecutionResult;
import org.junit.platform.launcher.TestExecutionListener;
import org.junit.platform.launcher.TestIdentifier;

import java.io.IOException;
import java.io.PrintWriter;

public class MyTestExecutionListener implements TestExecutionListener {
    // implementation
	 
    private PrintWriter writer;

    public MyTestExecutionListener() {
        try {
            writer = new PrintWriter("my-test-report.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void executionStarted(TestIdentifier testIdentifier) {
        if (testIdentifier.isTest()) {
            writer.println("STARTED: " + testIdentifier.getDisplayName());
        }
    }

    @Override
    public void executionFinished(TestIdentifier testIdentifier, TestExecutionResult result) {
        if (testIdentifier.isTest()) {
            writer.println("FINISHED: " + testIdentifier.getDisplayName() + " [" + result.getStatus() + "]");
            writer.flush();
        }
    }

    @Override
    public void executionSkipped(TestIdentifier testIdentifier, String reason) {
        writer.println("SKIPPED: " + testIdentifier.getDisplayName() + " (" + reason + ")");
        writer.flush();
    }	 
}
```

The code is basic, it overrides three of the eight default methods of TestExecutionListener and uses testIdentifier to retrieve @DisplayName values. It prints them to a textfile in the root of the project and I got this output (don't give it interpretation, the test was a silly one fiddling with @CsvSource).

```
#my-test-report.txt

STARTED: [1] apple, 1
FINISHED: [1] apple, 1 [SUCCESSFUL]
STARTED: [2] banana, 2
FINISHED: [2] banana, 2 [SUCCESSFUL]
STARTED: [3] lemon, lime, 0xF1
FINISHED: [3] lemon, lime, 0xF1 [SUCCESSFUL]
STARTED: [4] strawberry, 700_000
FINISHED: [4] strawberry, 700_000 [SUCCESSFUL]
```

### Under the hood

While making this work I wondered how the TestIdentifier argument was injected in the code. The three methods in MyTestExecutionListener have a TestIdentifier object as argument but someone must have called the methods with a value for this argument and it wasn't me. I asked ChatGPT and it gave a good answer, of which this are the final four bulletpoints:

1.	Launcher discovers and registers your TestExecutionListener.
2.	Launcher builds a TestPlan using all discovered test engines.
3.	Each test element (method, class) becomes a TestIdentifier.
4.	As each test starts/finishes, JUnit calls your listener methods and injects the relevant TestIdentifier.

The involved internal classes that do the invocations are:

- org.junit.platform.launcher.core.DefaultLauncher
- org.junit.platform.launcher.core.EngineExecutionOrchestrator
 
ChatGPT describes this as a classic _observer_ pattern. Java has a default Observer interface, java.util.Observer, that has an update() method. It can be implemented by Observers, and the call to update() is made by a so called _Observable_. 
In this case the TestExecutionListener is the observer and the observables must be the classes mentioned above. They are the central hub that broadcast the required information to listeners. 


