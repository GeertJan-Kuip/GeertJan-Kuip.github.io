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

- **@ExtendWith(SpringExtension.class)** +** @ContextConfiguration** = **@SpringJUnitConfig**
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

- AC means Application Context for convenience
- Three types: 
    - Full test (full application context + server)
    - MockMVC test (full application test, no server)
    - Slice testing (partial application context, no server)
- **@SpringBootTest** loads same AC as application. It searches for **SpringBootConfiguration**,  annotation of @SpringBootApplication.
- In slice testing, the mocking of those part of the AC that is not generated, is done by **@MockBean**





## Section 5 - Security

## Section 6 - Spring Boot



