# Maven and testing with JUnit 5

### Basic pom configuration

The Surefire plugin detects test classes, loads a test engine and delegates test execution to that engine. Surefire is available by default, JUnit or TestNG must be added as dependency. For JUnit it is almost straightforward:

```
      <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
      </dependency>
```

As you see verion number is omitted. JUnit advises to load their BOM (Bill of Materials) via ```<dependencyManagement>```. By doing so jupiter will inherit the right version number, just like other JUnit stuff you might add as dependency or plugin.

```
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.junit</groupId>
        <artifactId>junit-bom</artifactId>
        <version>5.13.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

### JUnit

JUnit is good for unit testing but you can do integration testing with it as well, I don't know everything yet but the annotations @ExtendWith and @RegisterExtension give a clue. You can create a special class that generates a context for a testclass, for example in the form of a webserver or a database, and at the same time manages the lifecycle of this context. 

Test classes must be put in src/test/java, using a copy of the directory structure of the src/main/java. Test classes must have a name starting or ending with Test, Maven will recognize this. When working with JUnit you will annotate test classes as well with @Test.

Test classes need the same package declarations as the non-test files, and the right imports from org.junit.jupiter. 

### JUnit annotations

JUnit has a set of annotations which I have categorized and described below. Together this gives an overview 


Annotations indicating testclass or -method

|    |    |
|----|----|
|@Test |Marks a test method  |
|@ParameterizedTest |Marks a test that runs multiple times. Requires a [source annotation](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources). There are many types of source annotations|
|@ParameterizedClass | Does the same for a whole class. The @Test methods in the class must all require the type of arguments that the source annotation provides | 
|@RepeatedTest |Does the same test multiple times. Besides n you can set parameters like 'failureThreshold'|
|@TestFactory |Marks methods that generate a stream, or some object that can be converted to a stream, of type ```<DynamicTest>``` or ```<DynamicNode>```. A DynamicTest object is composed of a display name (String) and a test of type ```<Executable>```, which is a functional interface representing a test. It typically has an 'assert' in its body. It throws a Throwable which makes it different from Runnable. DynamicNode is composed of a display name and an Iterable or Stream of ```<DynamicNode>``` objects. DynamicNode is the abstract parent class of both DynamicNode and DynamicTest. It means you can do nesting with it.|
|@TestTemplate |Marks a test that will be executed multiple times using a stream of ```<TestTemplateInvocationContext>``` instances. The test must be able to withstand multiple 'contexts'. The stream of TestTemplateInvocationContext objects is contained in a ```<TestTemplateInvocationContext>```, which you must create and then feed to the method annotated with @TestTemplate using '@ExtendWith.' The latter is the general way of providing contexts to methods.|
|@ClassTemplate ||

Annotations dealing with test execution order
@TestClassOrder
@TestMethodOrder

Whether to create separate class instance for every tested method in class
@TestInstance

Cosmetics
@DisplayName
@DisplayNameGeneration

Annotations for methods that should before or after method execution(s)
@BeforeEach
@AfterEach
@BeforeAll
@AfterAll
@BeforeParameterizedClassInvocation
@AfterParameterizedClassInvocation

Dealing with nested classes
@Nested

What test/method (not) to execute
@Tag
@DisAbled

Close resource after use for fields
@AutoClose

Provide context for more integrated tests
@TempDir
@ExtendWith
@RegisterExtension

