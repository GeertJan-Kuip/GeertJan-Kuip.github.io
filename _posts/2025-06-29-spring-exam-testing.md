# Spring exam - testing

## JUnit 5 annotations

JUnit 5 has major changes, annotation names from JUnit 4 have been replaced by new names:

- **@BeforeEach** (was @Before)
- **@BeforeAll** (was @BeforeClass)
- **@AfterEach** (was @After)
- **@AfterAll** (was @AfterClass)
- **@Disabled** (was @Ignore)

New annotations in JUnit 5:

- **@DisplayName**
- **@Nested**
- **@ParameterizedTest**

JUnit 5 are three components, JUnit 4 was one. The three packages are:

- JUnit Platform: foundation for launching testing frameworks on JVM
- JUnit Jupiter: extension model for writing tests and extensions in JUnit 5
- JUnit Vintage: testengine for running JUnit 3 & 4 tests on the platform

Make sure to import the JUnit 5 Jupiter API classes, not old stuff from JUnit 4.

## Integration testing

For unit tests you do not need Spring. Eventual dependencies need to be faked but you can do that with stubs and mocks.

JUnit 5 has extensible architecture via the @ExtendWith annotaion. It replaces JUnit 4's @RunWith. The advantage lies in the fact that with JUnit 5 you can use multiple extensions at the same time. Spring's extension point is the SpringExtension class, which is a Spring-aware test-runner.

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

So we could have used that in the previous code sample.

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








