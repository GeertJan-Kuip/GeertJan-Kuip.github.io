# Exam guide bullets

Next week I have planned to do the Spring Professional Develop (2V0-72.22) exam. I did some mock tests on Udemy and had a max score of 65%, which is not enough yet. Some of the questions on Udemy seemed to be a bit outdated, when I asked ChatGPT how to prepare it stressed to focus on the [exam guide](https://docs.broadcom.com/doc/vmw-spring-professional-develop-exam-guide). 

Here I will recreate the exam guide and provide bulletpoints for each topic mentioned. One of the difficult things is the sheer amount of concepts, classes and methods, making it hard to generate some overall picture in which every learnable element has its own distinct place. Hope this approach works.

## Section 1 - Spring Core

## Section 2 - Data Management

### 2.1 - Introduction to Spring JDBC

#### 2.1.1 Use and configure Spring's JdbcTemplate

- Simplifies JDBC operations. No need to manage connections, prepare statements, handle exceptions manually.
- JdbcTemplate needs a DataSource bean. Provide DataSource as argument. 
- DataSource and JdbcTemplate can be configured in Java-based style in @Configuration class.
- JdbcTemplate is itself not transactional, so you need @Transactional to wrap methods that perform multiple DB operations.
- You can field-inject JdbcTemplate with @Autowired if you like.
- DataSource can be manually configured. Set driver, url, username, password.
- Url of datasource is something like "jdbc:h2:mem:testdb"
- Spring Boot can autoconfigure DataSource if it can find appropriate database dependency. You do not have to mention DataSource anymore, nor inject it in JdbcTemplate.
- For autoconfiguration of DataSource, set driver, url, username, password in some properties file or environment or wherever.
- Use Jdbc if you need fine-grained control, performance, minimal dependencies or when you work with legacy databases. 
- Repositories or DAO's for Jdbc are beans with implemented methods.

#### 2.1.2 Execute queries using callbacks to handle result sets

- `.queryForObject(String query, Class<T> returntype)` returns one object.
- `.queryForObject(..)` can include bind variables as varargs: `.queryForObject(String query, Class<T> returntype, var1, var2, ..)`
- `.update(String query, [bind varargs])` can be used for INSERT, UPDATE and DELETE

- `.queryForMap(String query, [bind varargs])` returns `Map<String, Object>`
- `.queryForList(String query, [bind varargs])` returns `List<Map<String, Object>>`

- Mapping to a domain object can be done with RowMapper, ResultSetExtractor, RowCallbackHandler.
- These 3 are functional interfaces, their specific implementation provided as second argument. First argument is the query String as always.
- With these 3 you can still use bind variables as additional varargs argument.
- RowMapper and RowCallbackHandler are similar, the latter returns void which makes it better to stream the data result.

- RowMapper can be used with `.queryForObject(String query, RowMapper<T> rowMapper)` for one result or with `.query(String query, RowMapper rowMapper)` for multiple results, returned in list. The latter iterates automatically.
- ResultSetExtractor uses '.query(String query, ResultSetExtractor<T> rse)` and can be used to extract multiple rows into one object. You typically iterate over the resultset manually with a `while` loop. Good fine-grained control, more freedom to customize.

#### 2.1.3 Handle data access exceptions

- JDBC throws SQLException by default
- Bad traits: it is checked, provides no useful information, too candid about db type and vendor
- Problem with checked: must be thrown along the whole call stack.
- Therefore Spring throws DataAccessException: unchecked (extends RunTimeException), meaningful child exceptions, not candid about database in messaging
- Five children (there are more): 
    - DataAccessResourceFailureException 
    - CleanupFailureDataAccessException
    - OptimisticLockingFailureException
    - DataIntegrityViolationException
    - BadSqlGrammarException

### 2.2 - Transaction Management with Spring

#### 2.2.1 Describe and use Spring Transaction Management

- Used for database operations
- ACID: Atomic, Consistent, Isolated and Durable
- Setup requires 3 steps:
    - Declare a PlatformTransactionManager Bean
    - Declare the transactional methods with annotation or programmatic
    - Add @EnableTransactionManagement to a configuration class
- PlatformTransactionManager is an interface, implementations are DataSourceTransactionManager, JpaTransactionManager and others.
- @Transactional can be added to method or class, in latter case all methods in class are transactional.

- These are the 5 isolation levels in trtansactions (attribute 'isolation'):
    - DEFAULT
    - READ_UNCOMMITTED
    - READ_COMMITTED  // most often the default
    - REPEATABLE_READ
    - SERIALIZABLE
- Apart from default, which is dependent on the database, the order is from loose to strict. READ_UNCOMMITTED is loose, SERIALIZABLE most strict.
- Relevant terms: 
    - Dirty Read: reading uncommitted data from another transaction
    - Non-repeatable Read: getting different results for the same query in the same transaction
    - Phantom Read: A query returns different sets of rows due to new inserts by others

#### 2.2.2 Configure Transaction Propagation

- @Transaction(propagation=Propagation.REQUIRED) is the default (only one transaction)
- REQUIRES_NEW creates a new independent inner transaction and suspends the outer.
- This means that failure and success of the different nested transactions is independent.
- When a method calls a @Transactional(propagation=Propagation.REQUIRES_NEW) annotated method in the same class, its propagation rule is not followed because no proxy is created for inner class calls. 
- General rule: Java makes it impossible for inner calls to be intercepted (and thus make a proxy).

#### 2.2.3 Setup Rollback rules

- By default transactions will not be rolled back because of a checked exception. Only unchecked exceptions will cause a rolback.
- Thus it is fortunate that Spring always uses unchecked exceptions.
- Rollback behaviour can be customized with the rollbackFor and noRollbackFor annotation. Provide a Class<T> as argument, must be an Exception class. So you can let a specific RuntimeException cause no rollback, or a specific checked exception cause a rollback:
    - @Transactional(rollbackFor=MyCheckedException.class)
    - @Transactional(noRollbackFor={MyOtherUncheckedException.class, MySecondUncheckedException.class})

#### 2.2.4 Use Transactions in Tests

- Adding @Transactional to a @Test annotated class will make the transaction being rolled back
- This only applies to the outer transaction, not to an inner transaction!
- If you add @Commit to a test or class annotated with @Test and @Transactional prevents the rollback

### 2.3 - Spring Boot and Spring Data for Backing Stores

#### 2.3.1 - Implement a Spring JPA application using Spring Boot

- JPA can be used by Spring Boot or Spring Framework
- In Spring Framework, you need to manually configure:
    - DataSource, 
    - EntityManagerFactory, 
    - JpaTransactionManager, 
    - and add @EnableJpaRepositories
- In Spring Boot, everything is autoconfigured. You need to add the @Entity annotations to domain classes and add the `spring-boot-starter-data-jpa` dependency.
- @EntityScan("other.package") customizes where to search for domain classes. Note: it will _only_ scan other.package now.
- application.properties requires some settings to guide autoconfiguration
    - `spring.jpa.database=default` is recommended 
    - `spring.jpa.hibernate.ddl-auto=xxx` is important. Options: `validate | update | create | create-drop`
    - `spring.jpa.show-sql=true` + `spring.jpa.properties.hibernate.format-sql=true` show SQL being run in log, well-formatted
    - `spring.jpa.properties.hibernate.xxx=???` sets any Hibernate property

#### 2.3.2 - Create Spring Data Repositories for JPA

- You need domain classes annotated with @Entity (and things like @Id, @Table). They will be auto-scanned by Spring Boot.
- DB table name will be class name unless you annotate class with @Table("otherName")
- The field representing a key must be of Long type. Annotate with @Id and GeneratedValue(strategy=GenerationType.AUTO)
- Other values for strategy:
    - GenerationType.IDENTITY
    - GenerationType.SEQUENCE
    - GenerationType.TABLE

- Create a repository by extending interface `Repository<T, ID>` or a child of it. Those are:
    - CrudRepository (has CRUD methods)
    - PagingAndSortingRepository (CRUD + Iterable, Page, fiindAll)
    - JpaRepository (has everything plus more)
- No method implementations required
- Method names are translated to db operations
- Unless you add @Query("some query") to a method. This allows for finer grained control/complex queries. Can be used with placehiolders.
- Spring Boot scans for these repository interfaces in its own package and below
- Change places to scan with `@EnableJpaRepositories(basePackages="com.jan.repository")` added to @Configuration class.
- Repository interfaces are proxied and these proxies are beans that can be injected as arguments, for example in a service bean. No @Autowired just by the name. Good practice.

- Valid starters of method names (there are more), see [link](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.query-methods.query-creation):
    - findBy
    - findTopBy
    - queryFirst10By
    - findTop3By
    - findFirst10By
    - findDistinctPeopleBy
    - findPeopleDistinctBy
    - findByLastnameOrFirstname
    - findContainingEscaped

## Section 3 - Spring MVC

### 3.1 - WebApplications with Spring Boot

#### 3.1.1 Explain how to create a Spring MVC application using Spring Boot

- Web Servlet vs  Web   Reactive/Flux. The latter is newer, non-blocking
- `spring-boot-starter-web` is the basic dependency for MVC application
- A Spring MVC application typically has:
    - A @Controller or @RestController class
    - @RequestMapping or similarly annotated methods
    - A `Model` onject or JSON responses
    - A view resolver (Thymeleaf, JSP etc.)
- Spring does much of the work, you need to create controllers and views
- @Controller + @ResponseBody = @ RestController
- To make single method REST: `public @ResponseBody List<Account> list() {...}`
- Handler adaptor interpretes the request and creates beans and variables you can work with. Examples:
    - @PathVariable
    - @RequestParam
    - @RequestHeader
    - @RequestBody
    - Principal
    - Locale
    - HttpMethod
    - HttpSession
    - HttpServletRequest
- See [examples](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-08-spring-boot-web-application.md#the-handler-adapter1)
- `@RequestParam(required=false)` allows for optional request parameter. Can return null, so type cannot be primitive.
- Client indicates what content type he wants with the 'Accept' header. `application/xml` and `application/json` indicates xml or json
- Spring reads Accept header and invokes appropriate HttpMessageConverter
- To build a customized response, including headers, use [ResponseEntity](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-08-spring-boot-web-application.md#customizing-response-with-responseentity). You can build it in a chain.

#### 3.1.2 Describe the basic request processing lifecycle for REST requests

- Http request enter the DispatcherServlet
- DispatcherServlet delegates to:
    - HandlerMapping
    - HandlerAdaptor (wraps Controller)
    - MessageConverter (like Jackson)
    - ViewResolver (like Thymeleaf, not for  REST)
- DispatcherServlet sends back the response

#### 3.1.3 Create a simple RESTful controller to handle GET requests

- See 3.1.1
- Use @GetMapping with value attribute that defines what url's to handle
- Or use @RequestMapping with `method=RequestMethod.GET`
- PATCH, HEAD, OPTIONS and TRACE can also be selected as method.
    - PATCH partially updates a resource
    - HEAD is like GET but returns only headers
    - OPTIONS describes what HTTP methods and features are supported by the server
    - TRACE echoes the received request for debugging purposes
- Placeholders in the value arg of GetMapping, check [post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-08-spring-boot-web-application.md#extracting-path-elements) 

#### 3.1.4 Configure for deployment

- By default Spring Boot starts up an embedded webserver, TomCat
- To change Tomcat, use `<exclusions>` section in `spring-boot-starter-web` and add alternative server as dependency, like `spring-boot-starter-jetty`.
- If you want app to run in existing web container, you need to adjust the Spring Boot application class: 
    - `public class Application extends SpringBootServletInitializer`
    - Swap `main` method for `configure` method
- Hybrid approach (does both JAR and WAR):
    - `public class Application extends SpringBootServletInitializer {`
    - add both a `main` method for JAR and a `configure` method for WAR. See [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-08-spring-boot-web-application.md#the-hybrid-variant)
- WAR can be run stand-alone if Tomcat or Jetty is embedded. If you have such a WAR file, mark the Tomcat/Jetty dependencies `provided` for the case where the app will be embedded in existing server. This prevents version conflicts between servers. 
- Spring Boot creates fat and skinny jar and also war. Requires `spring-boot-maven-plugin`
- Spring Boot Developer Tools: add dependency `spring-boot-devtools`
- Creates two separate classloaders, decreases startup time as libraries won't be reloaded every time. Feature automatically disabled if Spring thinks app is running in 'production'

### 3.2 - REST Applications

#### 3.2.1 Create controllers to support the REST endpoints for various verbs

- MessageConverters do ORM both ways
- The return value of a controller method is the body of the response. Use void for empty body

- [@PutMapping](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-11-get-put-post-delete-resttemplate.md#putmapping)
- The @ResponseStatus annotation on method lets you respond specific status, like in @ResponseStatus(HttpStatus.NO_CONTENT) that returns 204 with no body.
- Access body content of request with @RequestBody before method argument
- Type of body (say json) is converted to domain object immediately, automatically
- This decoupling of web- and application environment makes testing much easier.

- [@PostMapping](@ResponseStatus(HttpStatus.NO_CONTENT))
- This one is difficult because the URI of the location where item is stored needs to be sent back under the 'Location' header
- PostMapping returns `201 Created` upon success, empty body

- [@DeleteMapping](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-11-get-put-post-delete-resttemplate.md#deletemapping)
- You typically return `204 No Content`
- Can use `@ResponseStatus(HttpStatus.NO_CONTENT)` for that 

#### 3.2.2 Utilize RestTemplate to invoke RESTful services

- [RestTemplate](@ResponseStatus(HttpStatus.NO_CONTENT)) lets you build requests to other servers. It needs RequestEntity to customize headers. 
- GET, POST, PUT, DELETE correspond to the following RestTemplate methods:
    - getForObject -> returns Object (MessageConverter at work)
    - postForLocation -> returns URI
    - put
    - delete 
- Use `.getForEntity` instead of `.getForObject` to get access to the whole response, including headers etc. See [link](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-11-get-put-post-delete-resttemplate.md#access-response-headers-with-responseentity). 
- `.getForEntity` returns a ResponseEntity<T> object. Methods:
    - getStatusCode()
    - getHeaders()
    - getBody() -> returns object, using MessageConverter
- [RequestEntity](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-11-get-put-post-delete-resttemplate.md#customizing-requests-with-requestentity) is the counterpart of ResponseEntity. Used by RestTemplate to customize headers etc of request


## Section 4 - Testing

### 4.1 - Testing Spring Applications

#### 4.1.1 Write tests using JUnit 5

- [Here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-29-spring-exam-testing.md) is the article
- Testing is done with JUnit, Spring gets involved when annotations need to be processed and when (parts of) the application context are required (integration testing)
- JUnit 5 annotation commonly  used are:
    - **@BeforeEach** (was @Before)
    - **@BeforeAll** (was @BeforeClass)
    - **@AfterEach** (was @After)
    - **@AfterAll** (was @AfterClass)
    - **@Disabled** (was @Ignore)
- Be aware that @Before, @BeforeClass, @After, @AfterClass, @Ignore are JUnit 4. Not used anymore.
- Remember by: _**Each and All, Disabled. No Class, always a postfix.**_

- Other important JUnit 5 annotations are:
    - **@ExtendWith** (was @RunWith in JUnit 4)
    - **@DisplayName**
    - **@Nested**
    - **@ParameterizedTest**

#### 4.1.2 Write Integration Tests using Spring

-  The relevant Spring annotations are:
    - **@ContextConfiguration**
    - **@SpringJUnitConfig**
    - **@TestPropertySource**
    - **@DirtiesContext**
    - **@ActiveProfiles**
    - **@Sql**

- **@ExtendWith(SpringExtension.class)** + **@ContextConfiguration** = **@SpringJUnitConfig**
- Give **@SpringJUnitConfig** a value attribute type Class that points to relevant configuration class/bean
- JUnit 5 has Platform, Jupiter, Vintage

- **@DirtiesContext** is used to reload a fresh application context after the test, because its state might be compromised
- **@TestPropertySource** imports properties in files or directly. Higher precedence than other properties. Highest for values directly set as attribute value.
- **@TestPropertySource** looks automatically for a properties file with the same name as the testclass.
- **@ActiveProfiles** sets which profiles you want to use. Automatically includes all beans that belong to no profile.

#### 4.1.4 Extend Spring Tests to work with Databases

- Use **@Sql** to prepare a database to be used during the test
- Spring looks in the application context of your test what database to use. It is found in the DataSource bean.
- Spring Boot autoconfigures DataSource, based on application.properties or application-test.properties, if active profile is test
- Spring Boot configures in-memory db if none is found
- If multiple DataSources are available, mark one with @Primary. Or use **@SqlConfig(dataSource="..")**
- The value/scripts annotation lets you define .sql script files to run before the test (create tables etc, load some data)
- The 'executionPhase' lets you set whether .sql file is run before or after the method (allows for cleanup)
- Example: `@Sql(scripts = "/testfiles/cleanup.sql", executionPhase=Sql.ExecutionPhase.AFTER_TEST_METHOD)`
- @Sql can be added to class and method (easier to customize) 
- In attribute you can tell what to do when script fails:
    - FAIL_ON_ERROR
    - CONTINUE_ON_ERROR
    - IGNORE_FAILED_DROPS
    - DEFAULT
- In attribute you can as well set something about syntax control: comments, statement separator
- Example: `@Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`"))`

- [More info](https://docs.spring.io/spring-framework/reference/6.2-SNAPSHOT/testing/testcontext-framework/executing-sql.html#testcontext-executing-sql-declaratively)

### 4.2 - Advanced Testing with Spring Boot and MockMVC

- Here, AC means Application Context for convenience

#### 4.2.1 Enable Spring Boot Testing

- Add `spring-boot-starter-test` with scope 'test' in pom
- This starter is included automatically if you load a Spring Boot project from start.spring.io
- Included dependencies are:
    - JUnit
    - Spring Test & Spring Boot Test
    - AssertJ
    - Hamcrest
    - Mockito
    - JSONassert
    - JsonPath
- **@SpringBootTest** loads same AC as application. It searches for **@SpringBootConfiguration**,  annotation of @SpringBootApplication.
- Use **@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)** for full integration test. Or DEFINED_PORT.
- Use WebEnvironment.MOCK for MockMVC testing
- There is also WebEnvironment.NONE

#### 4.2.2 Perform integration testing

- Integration testing is organized around **TestRestTemplate** (no child of RestTemplate)
- TestRestTemplate builds and sends HTTP requests to the application
- TestRestTemplate is autoconfigured if you use `RANDOM_PORT` or `DEFINED_PORT`
- Use RestTemplateBuilder to build a TestRestTemplate HTTP request
- Methods:
    - .postForLocation(..) -> returns URI
    - .getForObject(..) -> returns Object
    - .getForEntity(..) -> returns ResponseEntity
    - .delete(..) -> returns void
- Good traits of TestRestTemplate:
    - Takes a relative path instead of an absolute
    - Does not throw error when an error response (such as 404) is thrown from server
    - Configured to ignore cookies and redirects

#### 4.2.3 Perform MockMVC testing

- Server not running, no ports involved. Requests through DispatcherServlet.
- Use `@SpringBootTest(webEnvironment=WebEnvironment.MOCK)`
- Central element is the MockMVC bean, which is autoconfigured
- Use `@AutoConfigureMockMvc` to get autoconfigured MockMVC bean
- Inject MockMVC bean in @Test class with @Autowired on field
- MockMVC uses the `.perform(RequestBuilder requestBuilder)` method to do the mock request
- Difficulty lies in generating the HTTP request
- Use [static imports](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-13-spring-boot-testing-overview-testresttemplate.md#use-static-imports) 
- Static imports provide the methods of MockMvcRequestBuilders and MockMVCResultMatchers
- The `.get()`  method of MockMvcRequestBuilders returns a MockHttpServletRequestBuilder object which allows for chain build, adding content, headers etc.
- No standard assert-like code is required, the testing is done in the .andExpect() method.
- Click [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-13-spring-boot-testing-overview-testresttemplate.md#mockmvc-object) to get more detailed explanation

#### 4.2.4 Perform slice testing

- Do not use @SpringBootTest
- Instead use annotations for partial AC loading like:
    - **@WebMvcTest @WebFluxTest**
    - **@DataJpaTest @DataJdbcTest @JdbcTest**
    - **@DataMongoTest @DataRedisTest**
- The attribute for those anotations is the class where the configuration for it is found. For @WebMVCTest this is the @Controller or @RestController annotated class. See [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-13-spring-boot-testing-overview-testresttemplate.md#slice-testing-mvc-with-webmvctest)
- These annotations result in the autoconfiguration of a MockMVC bean, just like in MockMVC testing
- Inject MockMVC bean with @Autowired in your test class
- @MockBean is the central annotation. Use it on a field that declares an object of a type that is a bean in the AC but not in the sliced AC of the test. Now it will be included as a mock.
- @MockBean is a Spring annotation, @Mock is a Mockito annotation.
- In the @Test class, define how the mock should behave or what value it should return on a specific request
- Now use `mockMVC.perform(..)` exactly in the way you would use it in MockMVC testing

- If you want to test the repository, you can use @DataJpaTest
- Only the @Repository beans will be loaded
- Spring will autoconfigure a TestEntityManager object, that can be used instead of EntityManager
- TestEntityManager provides a subset of EntityManager's methods
- By default, an in-memory database is used to replace the production db
- To specify the db yourself, use the **@AutoConfigureTestDatabase** annotation

## Section 5 - Security

### 5.1 - Explain basic security concepts

Major concepts
- Principal
- Authentication
- Authorization
- Authority
- Secured Resource

Authentication mechanisms:
- Basic
- Digest
- Form
- X.509
- OAuth 2.0 / OIDC

Storage options for credential and authority data:
- in-memory (development only)
- Database
- LDAP

Why Spring Security?

- **Portable**
    - Can be used on any Spring project
- **Separation of Concerns**
    - Business logic is decoupled from security concern
    - Authentication and Authorization are decoupled
- **Flexible and Extensible**
    - Choose Authentication method yourself
    - Choose storage method yourself
    - Highly customizable

The Schema according to video tutorial:
- Security Context delegates to Authentication Manager
- Access to Secured Resource is intercepted by Security Interceptor
- Security Interceptor consults Configuration Attributes and delegates to Authorization Manager

Three steps for Setup and Configuration
- Setup filter chain
- Configure securtity (authorization) rules
- Setup Web Authentication

### 5.2 - Use Spring security to confugure Authentication and Authorization

- Step 1 is to set up a filter chain
- Requires a DelegatingFilterProxy, which is autoconfigured by Spring Boot. To match the different life cycles. 
- DelegatingFilterProxy is not a Spring Security class but a Spring MVC class. All requests pass through it.
- It delegates to SpringSecurityFilterChain, that knows which filters to call by what method in what order. 
- Only after the filtering is done, DelegatingFilterProxy sends the request to DispatcherServlet.
- The reponse will take the same route back, again through all the filters.

Most important filters:
- **SecurityContextPersistencefilter** - Establishes SecurityContext and maintains between HTTP requests
- **LogoutFilter** - Clears SecurityContextHolder when logout requested
- **UsernamePasswordAuthenticationFilter** - Puts Authentication into the SecurityContext on login request
- **ExceptionTranslationFilter** - Converts SpringSecurity exceptions into HTTP response or redirect
- **AuthorizationFilter** - Authorizes web requests based on config attributes and authorities

More points:
- By default, Spring sets up security for all endpoints with the same in-memory user name and password
- When setting up config class for security, create the filtercahin bean and the userdetailsservice bean (the AuthenticationManager)
- In filterChain bean, requestMatchers are used to provide different authorizations for different endpoints
- Use either Spring MVC matching rules, otherwise "Ant-style" pattern matching
- "/admin/*" only matches "/admin/xxx"
- "/admin/**" matches any path under admin
- requestMatchers chaining must (logically) go from most specific to least specific
- There are no predefined roles, you have to define them yourself

```
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	http.authorizeHttpRequests((authz)->authz
	  .requestMatchers("/signup", "/about").permitAll
	  .requestMatchers(HttpMethod.PUT, "/accounts/edit*").hasRole("ADMIN")
	  .requestMatchers("/accounts/**").hasAnyRole("USER", "ADMIN")
	  .anyRequest().authenticated());
	return http.build();
}
```

- Older code may use antMatchers/mvcMatchers. Deprecated in Spring Security 5.8
- requestMatchers decides which one to use, it uses mvcMatchers if MVC is in the classpath, otherwise antMatchers
- antMatchers is simpler than mvcMatchers, the latter can work with path variables and adjusts for servlet context path if your app is deployed under one.
- As said, both are deprecated

If you  want certain endpoints to be open-access and **bypass security**:

```
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
	return (web)-> web.ignoring().requestMatchers("/ignore1", "/ignore2");
}
```

- This is different from `.permitAll()` in the filterChain, as these endpoints will still have to go through the filterchain.

Authentication flow
- It starts with BasicAuthenticationFilter, one in the filter chain
- It extracts a UsernamePasswordAuthenticationToken from the request
- It delegates to AuthenticationManager/ProviderManager
- Who delegates to some AuthenticationProvider
- Who delegates to some UserDetailsService
- Who tries to load the user and returns a UserDetails object if so
- If not, it throws an Exception
- The UserDetails object is stored in the SecurityContext as Authentication, including authorizations
- The SecurityContext is wrapped in SecurityContextHolder, who associates SecurityContext with the current execution thread

Out-of-the-box AuthenticationProviders:
- DaoAuthenticationProvider
- JaasAuthenticationProvider
- LdapAuthenticationProvider
- OpenIDAuthenticationProvider
- RememberMeAuthenticationProvider

Out-of-the-box UserDetailsService implementations:
- InMemoryUserDetailsManager
- JdbcUserDetailsManager
- LdapUserDetailsManager

You create an UserDetailsManager as a bean. Example:

```
@Bean
public InMemoryUserdetailsManager userDetailsService(){

	UserDetails user = User.withUsername("user").password(passwordEncoder.encode("user")).roles("USER").build();

	UserDetails admin = User.withUsername("admin").password(passwordEncoder.encode("admin")).roles("ADMIN").build();

	return new InMemoryUserDetailsManager(user, admin);
}
```

InMemoryUserdetailsManager is a class that implements UserDetailService and UserDetailManager interface. It is not suited for working with a database.

- JdbcUserDetailsManager is used for working with db
- Bean creation is easy
- There are default queries that Spring Security will execute to find the user but you can override them
- Support for groups is also available. Don't know what that means
- To implement custom authentication, you can implement custom UserDetailsService
- Or you can implement custom AuthenticationProvider
- Note: UserDetailService and AuthenticationProvider are the last two in the chain, the latter calls the former
- AuthenticationProvider has more information, namely the Authentication object. UserDetailsService has less information.

Password Encoding
- MD5PasswordEncoder and SHAPassWordEncoder are deprecated, cracked
- BcryptPasswordEncoder is the recommende encoder now

Challenges of Password Encoding Schemes:
- Should be future proof
- Should accomodate old password formats
- Should allow usage of multiple password formats

DelegatingPasswordEncoder stores passwords with a prefix that tells the hash algorithm. BCrypt is the current default.

- Add `.httpBasic(withDefaults())` to the SecurityFilterChain code to let the browser prompt the user for username and password
- To add login via form in website, use the `.formLogin(..)` and `.logout(..)` in the SecurityFilterChain code.
- Form login requires a html form in the webpage. The input field for username should have the name "username" and the field for password should have the name "password". The filter will recognize this as default values and handle it well.
- The form should send a POST request to "/login" and the login page should be "/login" itself.

### 5.3 - Define Method-level Security

- **@PreAuthorize** and **@PostAuthorize** are the annotations added to methods
- These two have a value attribute with some sort of SpEL expression indicating what the user should have (role, name, whatever) to be allowed access.
- This expression language has some methods/terms added by Spring Security
- **@EnableMethodSecurity** is the annotation added to a configuration class
- Spring Security uses AOP for method level security
- Recommendation 1: Secure your services
- Recommendation 2: Do not access other layers directly (ie do not bypass the service layer, this nullifies the security methods you place on them
- A secured method is wrapped as a proxy bean (AOP) and to access the methods, you will need to go past a Spring SecurityInterceptor, who delegates to the AuthorizationManager to see if the access should be granted.
- If unauthorized access is tried, it throws AccessDeniedException

## Section 6 - Spring Boot

### 6.1 - Spring Boot Feature Introduction

#### 6.1.1 Explain and use Spring Boot features

- Spring Boot
    - Takes opinionated view of the Spring platform and third-party libraries
    - Supports different project types like Web or Batch
    - Handles most low-level, predictable set-up for you
- Why Spring Boot?
    - Provide a radically faster and widely accessible getting-started experience for all Spring development
    - Be opinionated out of the box but get out of the way quickly as requirements start to diverge from the defaults
    - Provide a range of non-functional features that are common to large classes of projects
        - Embedded servers, metrics, health checks, externalized configuration, containerization, etc.

#### 6.1.2 Describe Spring Boot dependency management

- Spring Boot parent resolves versioning
- Spring Boot starters include all required transitive dependencies
- You can always overwrite: exclude dependencies you don't use, define the dependencies or their versions explicitly yourself
- Spring Boot parent pom does the following:
    - Defines versions of key dependencies
    - Defines Maven plugins
    - Sets up Java version

### 6.2 - Spring Boot Properties and Autoconfiguration

#### 6.2.1 Describe options for defining and loading properties

- Spring Boot will look for application.properties in:
    - in the config subdirectory of the working directory (`file:./config/`)
    - in the working directory itself `file:./`
    - in the config package in the classpath (`src/main/resources/config`)
    - in the classpath root (`src/main/resources`)
- Spring Boot will look for profile-specific property files: `application-{profile}.properties`. 
- application.properties will always be used, but profile specific files can override it
- In Spring Boot application.yml is accepted as alternative format, in regular Spring it won't work
- application.yml has the benefit that you can include profile-specific properties in it. No need for separate files
- Precedence order of properties: see [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-04-spring-boot-properties.md#precedence). Note the highe precedence pf **@TestPropertySource** and **@SpringBootTest** properties

- **@ConfigurationProperties** can be used as an efficient replacement for **@Value**. It is aware of all properties in application.properties and their prefixes and utilizes this to map the properties on the fields of the class that have the right names.
- To make the properties bind to the fields and become available to the application context you can use the following strategies:
    - **@EnableConfigurationProperties** on the application class (main entry). Add Class argument.
    - **@ConfigurationPropertiesScan** on the application class (Spring Boot 2.2.0+)
    - **@Component** on the properties class itself
- In all three cases, the class annotated with @ConfigurationProperties becomes a bean. It must be a bean, otherwise no binding takes place. And if it is a bean, you can wire it to other beans and then its fields become accessible.
- See [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-04-spring-boot-properties.md#making-properties-available) for blog post on @ConfigurationProperties.
- **@ConfigurationProperties** utilizes relaxed binfing, which means that property names will be recognized even if they are written in a different nameing style (camel case, snake case, capitalized etc
-  This is to make it easier to work with properties from different sources, like SystemProperties, Windows environment variables etc.

#### 6.2.2 Utilize auto-configuration

- **@EnableAutoconfiguration** enables auto-configuration
- Spring boot automatically creates beans it thinks you need based on contents of class path, application context and configuration files.
- **@ComponentScan** has the attribute that tells where to find components. If **@SpringBootapplication**, then this annotation will have that attribute, named as 'scanBasePackages'.
- Spring Boot autoconfigures DataSource if in-memory db is on classpath.
- It will autocinfigure JdbcTemplate if (1) Starter JDBC is on classpath and (2) a DataSource is configured

#### 6.2.3 Override default configuration

- You can override Spring Boot's default configuration in the following ways:
    - Set some of Spring Boot's properties
    - Explicitly define beans yourself so Spring Boot won't
    - Explicitly disable some autoconfiguration
    - Change dependencies or their versions
- Blog post [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-07-04-spring-boot-auto-configuration.md)
- Spring properties can best be put in application.properties because this is read early
- Connection pool settings can be set to override the default settings: 
    - spring.datasource.initial-size = 
    - spring.datasource.max-active = 
    - spring.datasource.max-idle =
    - spring.datasource.min-idle =
- As autoconfiguration is based on type and not name, it is not important when overriding a bean what name you give it.
- Add 'exclude' attribute to **@EnableAutoConfiguration** or **@SpringBootApplication** to exclude autoconfiguration classes (and thus override default configuration):
    - `@EnableAutoConfiguration(exclude=DataSourceAutoConfiguration.class)`
    - `@SpringBootApplication(exclude=DataSourceAutoConfiguration.class)`
- You can disable as well via application.properties:
    - `spring.autoconfigure.exclude = org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration`
- If you want a different version of a dependency, set the desired version in its pom with a properties tag. It overrides the version set by the parent pom
- Changing one library for another can be done using the `<exclusions>` section under a dependency. Add the new dependency as a separate dependency. Main example: Jetty instead of Tomcat.

### 6.3 Spring Boot Actuator

- Actuator provides
    - Production grade monitoring without having to implement it yourself
    - A framework to easily gather and return metrics and health indicators
    - Integration with 3rd party monitoring system for aggregation and visualization
- Accessible via JMX
- Or as HTTP endpoints:
    - /actuator/info (empty by default, not exposed by default)
    - /actuator/health (default is minimal, status "UP")
    - /actuator/metrics (not exposed by default, allows deeper url paths)

#### 6.3.1 Configure Actuator endpoints

- requires `spring-boot-starter-actuator` dependency
- list of endpoints (not exhaustive):
    - beans
    - conditions - conditions used by autoconfiguration
    - env - properties in Spring environment
    - health
    - configprops - collated list of all @ConfigurationProperties
    - info
    - loggers - query and modify log levels
    - mappings - Spring MVC request mappings
    - metrics
    - session - fetch or delete user sessions (only if using Spring Session)
    - shutdown - shutdown gracefully, disabled by default
    - threaddump - performs a thread dump
    - jolokia - exposes JMX beans over HTTP (not just actuators)
- All endpoint except shutdown are enabled by default. There exist a bean for them.
- HTTP: Only health is exposed by default
- JMX: all enabled endpoints are exposed by default
- JMX can be enabled/disabled with property: `spring.jmx.enabled=true`
- HTTP capability only available when using Spring MVC, WebFlux or Jersey
- Change basepath actuator to 'admin' with `management.endponts.web.base-path=/admin`
- Actuator can be run on a different port as the application
- To expose endpoints, do:
    - `management.endponts.web.exposure.include=beans,env,info`
    - `management.endponts.web.exposure.include=*` (exposes all)

#### 6.3.2 Secure Actuator HTTP endpoints

- To secure an endpoint, we need to edit the SecurityFilterChain`bean. This can be done using the relative path or by using some classes/methods that are base-path agnostic. The latter prevents a scenario in which 'actuator' in the basepath is changed for another term, like 'admin'.

Example code:

```
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{

  http.authorizeHttpRequests((authz)->authz
	.requestMatchers("/actuator/health").permitAll()
	.requestMatchers("/actuator/**).hasRole("ACTUATOR")
	.anyRequest().authenticated());
  return http.build();
}
```
```
// base-path agnostic, preferred option
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

  http.authorizeHttpRequests((authz)->authz
	.requestMatchers(EndpointRequest.to(HealthEndpoint.class)).permitAll()
	.requestMatchers(EndpointRequest.toAnyendpoint()).hasRole("ACTUATOR")
	.anyRequest().authenticated());
  return http.build();
}
```

#### 6.3.3 Define custom metrics

- Spring Boot 2.0 uses Micrometer library (Multidimensional metrics, collected in vendor neutral way, then you can expose it in vendor specific format)
- Micrometer instruments your JVM-based application code without vendor lock-in (SLF4J for metrics)
- Designed to add little to no overhead to your metrics collection activity
- Four different types of metrics:
    - Counter
    - Gauge
    - Timer
    - DistributionSummary
- Classes are created or registered with a MeterRegistry bean
- Custom metric names are listed on the `/actuator/metrics` endpoint
- Custom metric data can be fetched at `/actuator/metrics/[custom-metric-name]`
- Two ways to access metrics data: Hierarchical vs Dimensional (This is about last part of url)
- **Hierchical metrics** uses sysem in which the name consists of subsequent key-value pairs separated by dot, like:
    - http.method.get.status.200
    - http.method.get.status.*
- method and status are keys, get and 200 are its respective values
- Difficult convention, often there is no hierarchy to decide the order. Adding new attributes can break queries.
- **Dimensional metrics**. Better way to achive the same. Metrics are tagged. Examples:
    - http?tag=method:get&tag=status:200
    - http?tag=method:get&tag=status:200&tag=region:us-east
- Characteristics: flexible, adding new attribute to query is easy. Order doesn't matter anymore. 

Creating a custom metric, doing it the complicated wat, in words: 
- There is a MeterRegistry bean that has methods to create a Counter, Gauge, Timer or DistributionSummary object
- Inject this MeterRegistry object in your @Controller class via the constructor
- `this.timer = registry.timer("orders.submit")' in your constructor sets the timer field to a Timer object that is registered under 'orders.submit'
- Your controller class must of course have the appropriate field for it (type Timer, Gauge, Controller or DistributionSummary)
- In a @RequestMapping-like method: wrap the method in the timer.record method, using a lambda.

Doing it the easy way:
- Use the @Timed annotation on a method with the name under which it is registered as attribute. No need to inject MeterRegistry or create a Timer object yourself.

Sample code, both ways:

```
public class OrderController{

  private Timer timer;

  public class OrderController(MeterRegistry registry){
	this.timer=registry.timer("orders.submit"));
  }

  @PostMapping("/orders")
  public Order placeOrder( ...){
	return timer.record(()-> { /* code placing an order ...*/});
  }

  @GetMapping("/orders")
  @Timed("orders.summary")  // all in one annotation
  public List<Order> orderSummary() {...}
}
```

- A Timer object provides count, mean, max and total of its metric
- Timer is part of MicroMeter

DistributionSummary works slightly different, with a builder:

```
@Controller
public class RewardController { 

  private final DistributionSummary summary;

  public RewardController(Meterregistry meter){
	summary = distributionSummary.builder("reward.summary")
				.baseUnit("dollars")
				.register(meter);
  }

  @PostMapping(value="/rewards")
  public RespomseEntity<Void> create(@RequestBody Reward reward) {
	summary.record(reward.amount);
  }
}
```

To get the metrics collected by the above sample, use `/actuator/metrics/reward.summary` 


#### 6.3.4 Define custom health indicators

- By default, you see only one metric under actuator/health
- Set `management.endpoint.health.show-details=always` to see more
- You can group health indicators in application.properties. Example:
    - `management.endpoint.health.group.mysystem.include=diskSpace,db`
- in the example, 'mysystem' is the name given to the group. diskSpace and db are known names for health metrics. Group name can be added as element to /actuator/health/
- Many health indicators setup automatically, provided their dependencies are on the classpath

Create custom health indicator:
- Create @Component class implementing Healthindicator interface. Override the health() method to return the status in the form of a Health type object
- Or extend the AbstractHealthIndicator and override the doHealthCheck() method
- Built-in statusses that can be returned are: 
    - DOWN
    - OUT_OF_SERVICE
    - UNKNOWN
    - UP
- You can add more details as well to the Health object, those will show up in the /health url

Integrate the actuator data in monitoring system, like:
- Atlas (Netflix)
- CloudWatch
- Datadog
- Dynatrace
- Ganglia
- Graphite
- InfluxDB
- JMX
- New Relic
- Prometheus
_ SignalFx
- StatsD
- Wavefront (VMware)









