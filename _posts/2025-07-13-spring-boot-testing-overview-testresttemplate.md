# Spring Boot Testing - Overview and TestRestTemplate

[Module 6](https://spring.academy/courses/spring-boot/lessons/spring-boot-testing-intro) of the Spring Boot video tutorials is about Spring Boot testing. Spring Boot testing builds on the Spring Testing framework, see a [previous post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-29-spring-exam-testing.md), but we learn a lot more. 


## General overview

The first video starts with an overview of the new Spring annotations that are used in Spring Boot testing. This is the list:

**@SpringBootTest**

**@WebMvcTest @WebFluxTest**

**@DataJpaTest @DataJdbcTest @JdbcTest**

**@DataMongoTest @DataRedisTest**

**@MockBean**

@SpringBootTest loads the same Application Context as the application does, which is convenient. It searches for @SpringBootConfiguration, which is an annotation of @SpringBootApplication. 

The other annotations are for so-called _sliced_ testing.

_**Use @SpringBootTest for integration testing and use @ContextConfiguration for slice testing.**_

Another general thing: you load the dependency _spring-boot-starter-test_ with scope _test_ in the pom file. It will be included automatically if you load a Spring project from start.spring.io. It brings in a whole list of (3rd party) dependencies, namely:

- JUnit
- Spring Test & Spring Boot Test
- AssertJ
- Hamcrest
- Mockito
- JSONassert
- JsonPath

There is support for different webEnvironment modes. webEnvironMent is an attribute of @SpringBootTest and you set it like `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`:

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

## Using MockMVC testing

The previous sample code had the Tomcat server running because of `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`. There is an alternative way to test endpoints whereby the internal server (or web container you might call it) is not being used. It is called MockMVC. It comes from spring-test.jar and processes its requests through the DispatcherServlet. No ports are involved.

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

You inject the MockMvc bean, which is autoconfigured because of the @AutoConfigureMockMvc annotation, in your test class and now you can use it using the `.perform(RequestBuilder requestBuilder)` method. The argument, RequestBuilder, is created using the static imports. The .perform(...) method returns a ResultActions object, on which you can apply methods like .andExpect(..). All in all you can make all sorts of chains to get the desired request and the desired test on the result. This is a typical one:

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







 

