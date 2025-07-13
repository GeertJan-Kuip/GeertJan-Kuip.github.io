# Spring exam - testing

Testing is done with JUnit and the whole course centers around JUnit version 5. Many of the test-specific annotations being used come from JUnit 5 and not from Spring. If it would be solely about unit testing and not integration testing, you could test a Spring application without using Spring, just using stubs and mocks for the dependencies.

So generally, it is good to be aware what part of the testing comes from Spring and what from JUnit. Spring's role is to provide the test classes with the content and configuration ressembling the application. You want the ApplicationContext to be used, you want to use application.properties or other configuration files, you want to activate profiles.

## JUnit 5 annotations

JUnit 5 has major changes, annotation names from JUnit 4 have been replaced by new names:

- **@BeforeEach** (was @Before)
- **@BeforeAll** (was @BeforeClass)
- **@AfterEach** (was @After)
- **@AfterAll** (was @AfterClass)
- **@Disabled** (was @Ignore)

New annotations in JUnit 5:

- **@ExtendWith**  (was @RunWith in JUnit 4)
- **@DisplayName**
- **@Nested**
- **@ParameterizedTest**

When using @ExtendWith, you typically add SpringExtension.class as argument to integrate with Spring. The second step to integrate with Spring, or your specific application, is to use Spring annotation @ContextConfiguration with the configuration class(es) of your application as argument(s). You can combine both annotations in @SpringJUnitConfig. 

For completeness, these are the Spring annotations to know (discussed below):

- **@ContextConfiguration**
- **@SpringJUnitConfig**
- **@TestPropertySource**
- **@DirtiesContext**
- **@ActiveProfiles**
- **@Sql**

## JUnit components

JUnit 5 has three components, JUnit 4 only one. The three packages are:

- JUnit Platform: foundation for launching testing frameworks on JVM
- JUnit Jupiter: extension model for writing tests and extensions in JUnit 5
- JUnit Vintage: testengine for running JUnit 3 & 4 tests on the platform

Make sure to import the JUnit 5 Jupiter API classes, not old stuff from JUnit 4.

## Integration testing

For unit tests you do not need Spring. Eventual dependencies need to be faked but you can do that with stubs and mocks.

JUnit 5 has extensible architecture via the @ExtendWith annotation. It replaces JUnit 4's @RunWith. The advantage lies in the fact that with JUnit 5 you can use multiple extensions at the same time. Spring's extension point is the SpringExtension class, which is a Spring-aware test-runner.

To set it up, start with:

```
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes={SystemTestConfig.class})
public class TransferServiceTests {

	@Autowired
	private TransferService transferService;

	@Test
	public void shouldTransferMoneySuccessfully() {
		TransferConfirmation conf = transferService.transfer(...);
	}
}
```

### Using @SpringJUnitConfig

@SpringJUnitConfig is a composed annotation that combines:

- @ExtendWith(SpringExtension.class) from JUnit 5
- @ContextConfiguration from Spring

So we could have used that in the previous code sample. As argument you can provide configuration classes, just as you would for @ContextConfiguration.

### Special use of @Autowired

Only in Spring testing you can use @Autowired for method arguments, like this:

```
@SpringJUnitConfig(SystemTestConfig.class)  // instead of @ExtendWith and @ContextConfiguration
public class TransferServiceTests{
	// @Autowired
	// private TransferService transferService;

	@Test
	public void shouldTransferMoneySuccessfully(@Autowired TransferService transferService){
		TransferConfirmation conf = transferService.transfer(...);
	}
}
```

### Overriding a bean

The tutorial gives an example in which a static inner class is used to customize settings for the specific enclosing testclass. The code is this:

```
@SpringJUnitConfig // Don't specify any config classes
public class JdbcAccountRepoTest {

	@Test
	public void shouldUpdateDatabaseSuccessfully(){...}

	@Configuration // looks for configuration embedded in this test class
	@Import(SystemTestConfig.class)
	static class TestConfiguration{
		@Bean
		public DataSource dataSource() {...} // overrides a bean with a test alternative
	}
}
```

### Recreating or not recreating ApplicationContext

When testing, the ApplicationContext required for the test is created once and then taken from cache for every subsequent test. This saves a lot of time and is okay if states of beans are not being modified in a way that subsequent tests suffer from it.

If you think that for some class it is better to have a newly created Apllication Context, not polluted by state changes made during previous tests, you can use this modifier on the test method of which you think that it pollutes the ApplicationContext:

```
	@Test
	@DirtiesContext  // this is the annotation that will result in new context AFTER this test is done
	public void someTestThatPollutesState(){
		// code
	}
```

The @DirtiesContext will force the closing and destruction of the ApplicationContext at the end of this test. A new ApplicationContext will be generated for the next test.

### Test Property Sources

Just like you have the @PropertySource annotation in production code to import new properties or new files containing properties, in the test environment you have the @TestPropertySource that does the same. 

If you provide a property directly as attribute, it will have precedence over a property that is in a file you provide. Both have precedence over other properties.

The default that @TestPropertySource will look for is [testclassname].properties

Example:

```
@SpringJUnitConfig(SystemTestConfig.class)
@TestPropertySource(properties = { "username=foo", "password=bar"}, locations = "classpath:/transfer-test.properties")
public class TransferServiceTests{
	...
}
```

## Using Spring Profiles in testing

In production code you use the annotation @Profile on a class or method to say that specific beans belong to that profile. The specific profile can be activated in the command line (using -D) or programmatically like `System.setProperty("spring.profiles.active", "embedded,production")`. Beans not belonging to any profile will be included in any profile.

In testing this annotation is being used:

```
@ActiveProfiles
```
It has the same function as activating a profile in production environment with -D or programatically. `@ActiveProfiles({"jdbc","dev"}) ` for example will activate all beans belonging to these profiles, plus all beans belonging to no profile. So basically this is the way you can decide what ApplicationContext your test will use.

## Testing and databases

It is normal in a test environment to replace 'real' databases with embedded databases. What you want is to prepare these temporary databases with a schema and content so there is something to test.

The `@Sql` annotation lets you select .sql files (these typically contain sql statements like create, insert etc) and those will be run before the testing starts. It is also possible to run them after the test, for example for cleanup. Example from tutorial:

```
@SpringJUnitConfig(...)
@Sql({"/testfiles/schema.sql","/testfiles/load-data.sql"})
public class MainTests{
	
	@Test  // schema.sql and load-data.sql run before this test
	public void success(){...}

	@Test
	@Sql(scripts = "/testfiles/setupBadTransfer.sql") // Runs before this method
	@Sql(scripts = "/testfiles/cleanup.sql", executionPhase=Sql.ExecutionPhase.AFTER_TEST_METHOD)  // runs after method
	public void transferError(){...}
}
```

@Sql has several attributes. You can configure:

- ExecutionPhase (will it run before or after method)
- What to do when script fails (FAIL_ON_ERROR, CONTINUE_ON_ERROR, IGNORE_FAILED_DROPS< DEFAULT)
- SQL syntax control: comments, statement separator








