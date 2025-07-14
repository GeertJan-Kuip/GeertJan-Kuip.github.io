# Spring Boot Testing

[Module 6](https://spring.academy/courses/spring-boot/lessons/spring-boot-testing-intro) of the Spring Boot video tutorials is about Spring Boot testing. Spring Boot testing builds on the Spring Testing framework, see a [previous post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-29-spring-exam-testing.md), but we learn a lot more. 

## General overview

There are three types of testing:

- Full testing: whole application context, running server
- MockMvc testing: whole application context, no server running
- Slice testing: partial application context, @MockBean, no server running.

### New annotations being used

The first video starts with an overview of the new Spring annotations that are used in Spring Boot testing. This is the list:

- **@SpringBootTest**
- **@WebMvcTest @WebFluxTest**
- **@DataJpaTest @DataJdbcTest @JdbcTest**
- **@DataMongoTest @DataRedisTest**
- **@MockBean**

@SpringBootTest loads the same Application Context as the application does, which is convenient. It searches for @SpringBootConfiguration, which is an annotation of @SpringBootApplication. 

### Slice testing

Instead of using @SpringBootApplication you can use @WebMvcTest, @DataJpaTest or a similar one to do so called 'slice testing'. With slice testing, you only initialize a specific part of the context, by providing one or more configuration classes as argument to the annotation. The other parts of the context, as far as they are required for your test, will need to be mocked by **@MockBean**.

### Dependency / pom

Another general thing: you load the dependency _spring-boot-starter-test_ with scope _test_ in the pom file. It will be included automatically if you load a Spring project from start.spring.io. It brings in a whole list of (3rd party) dependencies, namely:

- JUnit
- Spring Test & Spring Boot Test
- AssertJ
- Hamcrest
- Mockito
- JSONassert
- JsonPath

### WebEnvironment modes

There is support for different webEnvironment modes. Use RANDOM_PORT or DEFINED_PORT if a server will be running. WebEnvironMent is an attribute of @SpringBootTest and you set it like `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`:

- RANDOM_PORT
- DEFINED_PORT
- MOCK
- NONE

## TestRestTemplate

To test a web application, you want to connect to the ports with HTTP requests, testing the endpoints. You do not only want to know if the application acts correctly on well-formed requests, but also if it returns the right status codes on incorrect requests. As a test should not fail because the application returns an error status code, Spring Boot testing is created in a way that when you test an endpoint and expect some error status code, the test should pass, not fail.

TestRestTemplate (which is no child of RestTemplate) is the object you use to build and send HTTP requests to the application. You can use RestTemplateBuilder to build a TestRestTemplate HTTP request. 

Some good traits of TestRestTemplate:

- Takes a relative path instead of an absolute path
- Does not throw error when an error response (such as 404) is received from the server
- Configured to ignore cookies and redirects

### Sample code #1

This sample shows how to use @SpringBootTest, its attribute webEnvironment, and TestRestTemplate. TestRestTemplate is autoconfigured if you use RANDOM_PORT or DEFINED_PORT as attribute for @SpringBootTest:

```
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class AccountClientBootTests{

	@Autowired
	private TestRestTemplate restTemplate; 

	// test code
}
```

### Sample code #2

The sample below shows a test class where TestRestTemplate is used to test the endpoints of the application, more specifically using a GET, POST and DELETE request:

```
@Test
public void addAndDeleteBeneficiary(){

	String addUrl = "/accounts/{accountId}/beneficiaries";
	URI newBeneficiaryLocation = restTemplate.postForLocation(addUrl, "David", 1);
	Beneficiary newBeneficiary = restTemplate.getForObject(newBeneficiaryLocation, Beneficiary.class);
	
	assertThat(newBeneficiary.getName()).isEqualTo("David");

	restTemplate.delete(newBeneficiaryLocation);

	ResponseEntity<Beneficiary> response = restTemplate.getForEntity(newBeneficiaryLocation, Beneficiary.class);

	assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

## Using MockMVC - serverless testing

The previous sample code had the Tomcat server running because of `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`. There is an alternative way to test endpoints whereby the internal server (or web container you might call it) is not being used. It is called MockMVC. It comes from spring-test.jar and processes its requests through the DispatcherServlet. No ports are involved.

The difficulty with this one is creating the request, which requires some juggling with builder classes.

### Main annotations

Annotate the test class with the following two annotations:

```
@SpringBootTest(webEnvironment=WebEnvironment.MOCK)
@AutoConfigureMockMvc
```

### Use static imports

Both the video tutorial and the Spring documentation recoomends the use of static imports, to easily use Builder and Matcher methods. These are the imports:

```
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
```

### MockMvc object

You inject the MockMvc bean, which is autoconfigured because of the @AutoConfigureMockMvc annotation, in your test class. Just use @Autowired on a field. Now you can use it using the `.perform(RequestBuilder requestBuilder)` method. 

The argument, RequestBuilder, is created using the static imports. The .perform(...) method returns a ResultActions object, on which you can apply methods like .andExpect(..), which does the test. All in all you can make all sorts of chains to get the desired request and the desired test on the result. This is a typical one:

```
  @Test
  public void testBasicGet(){

	mockMvc.perform(get("/accounts")).andExpect(status.isOk());
  }
```

Note that the .get() method is actually `...MockMvcRequestBuilders.get`, while the .status() method is `...MockMvcResultMatchers.status`.

You can work with placeholders as well and a GET request can use both 'URI template style' and 'request parameter style':

```
mockMvc.perform(get("/accounts/{acctId}", "12345"))
  or
mockMvc.perform(get("/accounts?myParam={acctId}", "12345"))
```

### MockHttpServletRequestBuilder - headers and payload

As the .get() method returns a MockHttpServletRequestBuilder that has its own static methods, you can create a chain within the argument of .perform(). This allows for detailed request building. The `.andExpect(..)` method can be chained as well, its argument type is [MockMvcResultMatchers](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/result/MockMvcResultMatchers.html) and you are advised to do static import for its methods as well.

```
mockMvc.perform(get(  "/accounts/{acctId}", "12345").accept("application/json")  ).andExpect(...);

mockMvc.perform(get(  "/accounts/{acctId}", "12345").content("{ ... }")
				.contentType("application/json")  )   
				.andDo(print())  // add this line to print reult to console. Handy.
				.andExpect(...)				
				.andExpect(content().contentType("application/jjson"));
```

Some static methods provided by MockHttpServletRequestBuilder are:

|Method|Description|
|----|----|
|param|Add a request parameter - such as param("myParam", 123)|
|requestAttr|Add an object as a request attribute. Also, sessionAttr does the same for session scoped objects.|
|header|Add a header variable to the request. See also headers, which adds multiple headers.|
|content|Request body|
|contentType|Set content type (Mime type) for the expected response|
|accept|Set the requested type (Mime type) for the expected response|
|locale|Set the local for making requests.|

### MockMvcResultMatchers - assertions

The methods of MockMvcResultMatchers, imported statically, are found in the argument of `.andExpect(..)`. Some static methods provided by MockMvcResultMatchers are:

|Method|Description|
|----|----|
|content|Assertions relating to the HTTP response body|
|header|Assertions on the HTTP headers|
|status|Assertions on the HTTP status|
|xpath|Search returned XML using Xpath expression|
|jsonPath|Searh returned JSON using JsonPath|

## Slice testing

Slice testing means testing only a part of the application, mocking the other parts. No server is run and only part of the Spring application context is initialized. We thus cannot use @SpringBootTest, as this will initialize the whole context.

### Using @MockBean

The parts of the context that are not initialized but are nevertheless required for the test can be mocked using @MockBean. This is a Spring annotation, don't confuse it with @Mock, which comes from the Mockito framework. @MockBean lets you extend the sliced context so you have enough to work with.

### Slice testing MVC with @WebMvcTest

The code sample below illustrates a typical way of using @WebMvcTest. The argument of @WebMvcTest is the controller class you want to test, it will be initialized as 'sliced' context. Those parts of the context that are not initialized but are required to make things work are mocked with @MockBean, in this case the AccountManager object. The mock for accountManager will need to be created within a test method.

Note that @WebMvcTest autoconfigures a MockMvc bean, you can inject it with @Autowired. 

```
@WebMvcTest(AccountController.class)
public class AccountControllerBootTests {

	@Autowired
	private MockMvc mockMvc;

	@MockBean
	private AccountManager accountManager;

	@Test
	public void testHandleDetailsRequest() throws Exception {

		// test code, see below
	}
}
```

Here is the test method in which the AccountManager object is mocked:

```
	@Test
	public void testHandleDetailsRequest() throws Exception {
		
		// 'arrange'
		given(accountManager.getAccount(0L)).willReturn(new Account("12345", "John Doe"));


		// 'act and assert'
		mockMvc.perform(get("/accounts/0"))
			.andExpect(status().isOk())
			.andExpect(content().contentType(MediaType.APPLICATION_JSON)
			.andExpect(jsonPath("name").value("John Doe"))
			.andExpect(jsonPath("number").value("12345"));

		// 'verify'
		verify(accountManager).getAccount(0L);
	}
```

### Slice testing data access layer - @DataJpaTest

If you want to slice test the repository using @DataJpaTest, only the @Repository beans will be loaded, all other components won't. Spring will autoconfigure a TestEntityManager object, which is the test equivalent of EntityManager. TestEntityManager provides a subset of EntityManager methods, just those useful for tests.

By default, an embedded in-memory database is being used to replace any explicit or auto-configured DataSource. If you want to specify what database to use yourself, maybe the production db, use the @AutoConfigureTestDatabase annotation.










 

