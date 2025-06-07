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
|@ParameterizedTest |Marks a test that runs multiple times. Requires a [source annotation](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources). There are many types of source annotations.|
|@ParameterizedClass | Does the same for a whole class. The @Test methods in the class must all require the type of arguments that the source annotation provides. | 
|@RepeatedTest |Does the same test multiple times. Besides n you can set parameters like 'failureThreshold.'|
|@TestFactory |Marks methods that generate a stream, or some object that can be converted to a stream, of type _DynamicTest_ or _DynamicContainer_. A DynamicTest object is composed of a display name (String) and a test of type ```<Executable>```, which is a functional interface representing a test. It typically has an 'assert' in its body. It throws a Throwable which makes it different from Runnable. DynamicNode is composed of a display name and an Iterable or Stream of ```<DynamicNode>``` objects. DynamicNode is the abstract parent class of both DynamicNode and DynamicTest. It means you can do nesting with it.|
|@TestTemplate |Marks a test that will be executed multiple times using a stream of ```<TestTemplateInvocationContext>``` instances. The test must be able to withstand multiple 'contexts'. The stream of TestTemplateInvocationContext objects is contained in a ```<TestTemplateInvocationContextProvider>```, which you must create and then feed to the method annotated with @TestTemplate using '@ExtendWith.' The latter is the general way of providing contexts to methods.|
|@ClassTemplate |Same as @TestTemplate but here a class is created with methods in it that will all be tested with a context provided by ClassTemplateInvocationContextProvider, a class that has a 'provideClassTemplateInvocationContexts' method that returns a stream or iterable of ClassTemplateInvocationContext objects.|
|**EXECUTION ORDER**|     |
|@TestClassOrder| Marking class with this annotation allows you to define in which order nested classes will be executed. It is also possible to define the order of execution of first-order classes, it requires configuration in 'src/test/resources/junit-platform.properties' plus adding the @Order annotation to classes you want to order (or not, if you choose to order alphabetically for example).|
|@TestMethodOrder| Same as previous, but plays out one level lower. Apply this to the class (with OrderAnnotation.class as argument) and annotate the test methods in the class with @Order(Integer ordernumber).|
|**Keeping state**|     |
|@TestInstance| By default JUnit creates a new instance for every tested method in a class, to prevent that altered state by one method affects the outcome of a test of another method. If you want to have all methods be tested with a single class instance, annotate the class with @TestInstance(Lifecycle.PER_CLASS). A byeffect is that with one class instance you can use the @AfterAll and @BeforeAll annotations on non-static methods, or at least in a meaningful way.|
|**Readable output**||
|@DisplayName|Sets display name of test that will be displayed in test reports and by test runners and IDEs. Special characters and emojis allowed. You can combine it with @ParameterizedTest, @ValueSource and placeholder ({0}) to get readable sentences in your output.|
|@DisplayNameGeneration|Applied to classes, extra options to get better readable output. For example, can convert class names with hyphens to sentences without hypens.|
|**Before/After**||
|@BeforeEach|Denotes that the annotated method should be executed before each @Test, @RepeatedTest, @ParameterizedTest, or @TestFactory method in the current class.|
|@AfterEach|Denotes that the annotated method should be executed after each @Test, @RepeatedTest, @ParameterizedTest, or @TestFactory method in the current class.|
|@BeforeAll|Denotes that the annotated method should be executed before all @Test, @RepeatedTest, @ParameterizedTest, and @TestFactory methods in the current class. Methods must be static or class must be annotated with @TestInstance. |
|@AfterAll | Same as previous, only to be executed after all methods. |
|@BeforeParameterizedClassInvocation|Denotes that the annotated method should be executed once before each invocation of a parameterized class.|
|@AfterParameterizedClassInvocation | As previous but 'after'.|
|**Nested classes**||
|@Nested|Denotes that the annotated class is a non-static nested test class. Nesting is more relevant in testing than in regular Java code, provides opportunities for structured code and structured output.|
|**Filtering tests**||
|@Tag|Used to declare tags for filtering tests, either at the class or method level|
|@Disabled|Disables a test class or test method|
|**Closing resources**||
|@AutoClose|For fields. The built-in AutoCloseExtension automatically closes resources associated with fields. Field type must have a close() method, if not you can give an argument with the name of another method that must be executed to close the resource.|
|**Timeout**||
|@Timeout|Tells the amount of time a test has to succeed, otherwise makes it a fail.|
|**Context**||
|@TempDir|Injects a Path/File instance in field or method parameter if it requires so and cleans it up after execution. If your code does something with file creation, writing, reading, this helps cause there is a Path in which you can create it.|
|@ExtendWith|Basically about providing classes, methods, parameters, fields with a class/object that acts as a context. Enables integration testing. Good for simple, stateless contexts that can be used throughout the test suite.|
|@RegisterExtension|Similar, but used _within_ test class. Add it to a field that creates an object, and this object will become the context. More verbose but easier to configure because you can provide runtime arguments for the specific situation. @ExtendWith needs the class name that forms the context as an argument, so that class object is defined elsewhere without custom arguments. Better reusability but less flexibility.|

