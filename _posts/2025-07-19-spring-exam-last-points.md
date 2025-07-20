# Spring exam last points

This monday I have planned to do the VMWare Spring Exam. I'm slightly improving on the Udemy mock tests but I need a little more. ChatGPT is helping me out on topics I can improve and I'm summarizing it here in a quick and dirty way.

### @DataJpaTest

- loads:
    - @Repository classes
    - @Entity classes
    - EntityManager (or actually TestEntityManager, same role)
    - Transaction management (JpaTransactionManager)
    - JdbcTemplate !!!!!
- Sets up in-memory database, unless specified otherwise.
    - @DataJpaTest(properties="spring.datasource.url=jdbc:mysql://...")
    - @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
- Each test method is wrapped in a transaction that is rolled back after the test completes, so changes made during the test won’t persist.
- It scans only your domain/entity classes (and possibly converters, embeddables, etc.) — not controllers or services.
- When using @DataJpaTest with JUnit 5, it's not necessary to use @ExtendWith(SpringExtension.class)  because @DataJpaTest already contains it.
- The same is true for WebMvcTest.

### Initialization callbacks of a Spring bean

- There are three ways to define a bean method that is called upon initialization:
    - @PostConstruct
    - the .afterPropertiesSet() method, required if you implement InitializingBean interface 
    - the init method as defined either in .xml or as bean attribute (@Bean(initMethod = "init")
- The order in which these are called is the order in which they are listed here. It starts with @PostConstruct

### Destruction callbacks of a Spring bean

- Again three ways, executed in this order:
    - @PreDestroy
    - the .destroy() method of DisposableBean interface
    - the destroymethod as defined in xml or as bean attribute

- The BeanPostProcessors run either before or after initialization. What is meant by initialization is the three initialization callbacks.

### AOP

- The typical annotations (@Before, @AfterReturning etc) are called advices. The method they annotate is also named 'advice'.
- Aspect is the class annotated with @Aspect in which you find (multiple) advices.

### REST

- Rest is not a protocol, the protocol is http.
- Rest is interoperable and uses a uniform interface. It supports caching and is stateless.

### @TestConfiguration and @ContextConfiguration

- @TestConfiguration defines extra or overridden beans to be used while testing. 
- @TestConfiguration is commonly used on a static inner class.
- You can also use @Configuration on a nested static class, in this case the outer configuration will not be used at all. The inner class is now the configuration class for all tests in the outer class. See [blog post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-29-spring-exam-testing.md#overriding-a-bean)
- @ContextConfiguration tells which configuration classes from the production application to use. It is thus fairly different. @ContextConfiguration is part of SpringJUnitConfig.

### Spring Boot Actuator

- [This page](https://docs.spring.io/spring-boot/docs/2.1.11.RELEASE/reference/html/production-ready-endpoints.html) has a ton of info, although a bit old
- [This is the recent version}(https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)
- There was a question in mock test asking which endpoints existed. 
- Set all endpoints enabled: `management.endpoints.enabled-by-default=true`
- Set all endpoints exposed: `management.endpoints.web.exposure.include=*`
- You can change the default mapping of Health Indicator Statuses: 
    - `management.health.status.http-mapping.DOWN=501`
- You can change the logging level for the package by:
    - HTTP via POST to `/actuator/loggers/${LOGGER_NAME}`
- There are many endpoints, but three of them apply only to web applications (Spring MVC, Spring WebFlux, or Jersey): 
    - heapdump
    - logfile
    - prometheus

### Return NO_CONTENT 204

- @ResponseStatus(HttpStatus.NO_CONTENT)
- return ResponseEntity.noContent().build();
- return HttpStatus.NO_CONTENT;
- return new ResponseEntity<>(headers, HttpStatus.NO_CONTENT);
- response.setStatus(HttpServletResponse.SC_NO_CONTENT); (a bit odd)
- **Not possible**: Using the statusCode attribute on @RequestMapping. It doesn't have that attribute.
- You can place @ResponseStatus annotation on top of a class. All methods in class, unless specified otherwise, will return this status.

### @ConfigurationProperties

- Cannot be applied to field, only class. Same is true for @PropertySource
- The trick is to give it a prefix attribute: @ConfigurationProperties(prefix = "myapp")

### Transactions - propagation

These are the options:

|Value|Description|
|----|----|
|REQUIRED|Join an existing transaction if one exists; otherwise, start a new transaction. (Default)|
|REQUIRES_NEW|Always start a new transaction, suspending any existing transaction.|
|SUPPORTS|Join the current transaction if it exists; otherwise, execute non-transactionally.|
|NOT_SUPPORTED|Suspend any existing transaction and run non-transactionally.|
|NEVER|Must run non-transactionally; throws an exception if a transaction exists.|
|MANDATORY|Must join an existing transaction; throws an exception if none exists.|
|NESTED|Run within a nested transaction if a current one exists; otherwise, behave like REQUIRED.|

### CGLib proxy, JDK proxy and self-invocation

- Self invocation = inner method call
- Both CGLib and JDK proxy do not support this. They can't, as the Java thing does not intercept inner method calls.
- Thus neither CGLib nor JDK Proxy support self-invocation. It is a problem for all stuff working with proxies.

### Supported loggers

- [Spring Boot logging documentation](https://docs.spring.io/spring-boot/reference/features/logging.html)
- The supported loggers are:
    - Java Util Logging (bit simplistic)
    - Log4J2 (pretty good)
    - Logback (default, pretty good)
- Those are supported with a default configuration
- Spring Boot uses Commons Logging for all internal logging but leaves the underlying log implementation open.
- By default, if you use the starters, **Logback** is used for logging, being the implementation.
- In Spring Boot, default logging level is INFO
- TRACE and DEBUG are thus not shown.
- Change log level with `logging.level.root=`
- SLF4J is not a logging implementation, it is just an interface. Fits on all.
- Commons Logging is also an interface, it is used for internal logging.
- Log4J2 is a complete rewrite of Log4J with better performance, async logging, and safer configuration. Apache.

### Optional type as controller argument

- JDK 8’s java.util.Optional is supported as a method argument in combination with annotations that have a required attribute (for example, @RequestParam, @RequestHeader, and others) and is equivalent to required=false.
- This is a correct argument: `@RequestHeader(required=false) Optional<String> greeting` 
- (but only because the required=false attribute)

### Spring Boot devtools

- `spring-boot-devtools` is the name of the dependency
- [link to docs](https://docs.spring.io/spring-boot/reference/using/devtools.html)
- If your application is launched from java -jar or if it is started from a special classloader, then it is considered a “production application”. Devtools is enabled then.
- `-Dspring.devtools.restart.enabled=true` turns it on anyway
- Applications that use spring-boot-devtools automatically restart whenever files on the classpath change.
- DevTools works with two classloaders to speed up startup (only one needs reloading)

### Data types compliant with @RequestParam

You can add @RequestParam to the following types, and it will extract them from the url:
- String
- int
- `Map<String,String>`
- `List<Integer>`
- `MultiValueMap<String, String>`  (Spring collection, one-to-many key-value pairs)

### Possible annotations on controller method parameter

There are multiple annotations, some of them obscure:
- @MatrixVariable
- @CookieValue
- @RequestParam
- @RequestHeader
- @RequestBody
- @PathVariable
- @RequestPart
- @RequestAttribute
- @ModelAttribute
- @SessionAttribute
- @SessionAttributes

### Supported controller method arguments

See the [overview](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)

A list, without annotations like @RequestParam:
- Reader, Writer, ZoneId, Locale, HttpMethod, Principal, HttpSession, 
- PushBuilder, ServletRequest, ServletResponse, NativeWebRequest
- `HttpEntity<B>`, Model, ModelMap, RedirectAttributes, Errors, BindingResult
- SessionStatus, UriComponentsBuilder

You can use these as arguments (Principal principal), and Spring automatically provides them for you. 

### Autoconfiguration

- Types of @Conditional annotations:
    - **@ConditionalOnClass**: A specific class is on the classpath.
    - **@ConditionalOnMissingClass**: A class is not on the classpath.
    - **@ConditionalOnBean**: A specific bean is present in the context.
    - **@ConditionalOnMissingBean**: A specific bean is not present.
    - **@ConditionalOnProperty**: A specific property (like my.feature.enabled=true) is set.
    - **@ConditionalOnResource**: A resource (like a file) is available.
    - **@ConditionalOnWebApplication**: The application is a web app.
    - **@ConditionalOnNotWebApplication**: The app is not a web app.
    - **@ConditionalOnExpression**: A SpEL expression evaluates to true.
    - **@ConditionalOnJava**: A minimum Java version is detected.
    - **@ConditionalOnCloudPlatform**: Running on a specific cloud platform (like AWS or Kubernetes).

- Ordering annotations:
    - **@AutoConfigureOrder**(Ordered.HIGHEST_PRECEDENCE)
    - **@AutoConfigureBefore**(DataSourceAutoConfiguration.class)
    - **@AutoConfigureAfter**(DataSourceAutoConfiguration.class)

### JPA Repository method names

- [Query keyword reference](https://docs.spring.io/spring-data/jpa/reference/repositories/query-keywords-reference.html)
- [List of examples](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- There is always `By` in the name. These are the possible prefix parts:
    - find...By
    - read...By
    - get...By
    - query...By
    - search...By
    - stream...By
    - exists...By
    - count...By
    - delete...By
    - remove...By
- Informative examples:
    - ...OrderByFirstnameAscLastnameDesc (Asc and Desc)
    - findByLastNameAndFirstName  (Use 'And' if more criteria)
    - findByFirstname,findByFirstnameIs,findByFirstnameEquals  (all ok)
    - findByStartDateAfter  (After is ok)
    - findByAgeIsNull, findByAgeNull  (both ok)
    - deleteInBulkByRoleId
    - deleteByRoleId    
- Remarkable keywords:
    - Regex, MatchesRegex, Matches
    - IgnoreCase, IgnoringCase
    - AllIgnoreCase, AllIgnoringCase
    - Null, IsNul, NotNull, IsNotNull
    - StartingWith, IsStartingWith, StartsWith - 'Is' often an option
    - Exists
    - Distinct

### @Autowiring, @Qualifier and fatal exception

- By default, Spring resolves @Autowired entries by type. 
- If more than one bean of the same type is available in the container, the framework will throw a fatal exception.
- To resolve, use @Qualifier(name of the bean you want)\
- If you put @Autowiring on a final field, It will basically not compile. Error.
- What will happen if you use @Autowired on a beans collection, say `List<Foo>`? 
    - It is a collection so all same type
    - It will instantiate a collection filled with all the beans of the type
    - Docs: _It is also possible to provide all beans of a particular type from the ApplicationContext by adding the annotation to a field or method that expects an array of that type_

### GrantedAuthority - Spring Security user permissions

- **GrantedAuthority** is the interface in Spring Security that encapsulates high-level user permissions
- this is a correct way of annotating: `@PreAuthorize("hasRole('ADMIN')")`
- this too: `@PreAuthorize (“#username == authentication.name”)`
- Note: @PreAuthorize needs to evaluate a SpEL expression. It must be true or false.
- this is correct syntax: `@Secured("ROLE_ADMIN")`. Note: no SpEL on @Secured
- The difference between ROLE_ADMIN and ADMIN is something historical

### Spring Data JPA is transactional, Jdbc is not

- In Spring data most repository methods (e.g., save(), delete(), etc.) run within a transaction, either explicitly or implicitly.
- For read-only operations (like findById()), Spring will often use a transaction marked as read-only (for optimization), though this depends on configuration.
- Jdbc is not transactional. You have to apply the @Transactional annotation yourself.
- Jdbc is set by default to auto-commit=true.

### RequestMapping

- @RequestMapping can be applied at both class and method levels.
- By default, if the HTTP method is not specified, methods annotated with @RequestMapping are mapped to handle requests for all HTTP methods.
- There is no statuscode attribute in @RequestMapping.

### Idempotent Behavior of HTTP methods

- GET, HEAD, PUT, DELETE, OPTIONS, and TRACE are idempotent methods. You can safely repeat them, doesn't affect outcome.
- PATCH and POST do _not_ show idempotent behaviour.
- So only two methods are not idempotent, POST and PATCH.

### Default message converters

|Message Converter|What it does|
|----|----|
|**ByteArray**HttpMessageConverter|converts byte arrays|
|**String**HttpMessageConverter|converts Strings|
|**Resource**HttpMessageConverter|converts org.springframework.core.io.Resource for any type of octet-stream|
|**Source**HttpMessageConverter|converts javax.xml.transform.Source|
|**Form**HttpMessageConverter|converts form data to/from a MultiValueMap<String, String>|
|**Jaxb2RootElement**HttpMessageConverter|converts Java objects to/from XML (added only if JAXB2 is present on the classpath)|
|**MappingJackson2**HttpMessageConverter|converts JSON (added only if Jackson 2 is present on the classpath)|
|**MappingJackson**HttpMessageConverter|converts JSON (added only if Jackson is present on the classpath)|
|**AtomFeed**HttpMessageConverter|converts Atom feeds (added only if Rome is present on the classpath)|
|**RssChannel**HttpMessageConverter|converts RSS feeds (added only if Rome is present on the classpath)|

### Method Security

@EnableMethodSecurity has the following attributes:

|Return value|Attribute|Description|
|----|----|----|
|boolean|jsr250Enabled|Determines if JSR-250 annotations (**@RolesAllowed**) should be enabled.|
|...AdviceMode|mode|Indicate how security advice should be applied.|
|int|offset|Indicate additional offset in the ordering of the execution of the security interceptors when multiple advices are applied at a specific joinpoint.|
|boolean|prePostEnabled|Determines if Spring Security's **@PreAuthorize**, **@PostAuthorize**, **@PreFilter**, and **@PostFilter** annotations should be enabled.|
|boolean|proxyTargetClass|Indicate whether subclass-based (CGLIB) proxies are to be created as opposed to standard Java interface-based proxies.|
|boolean|securedEnabled|Determines if Spring Security's **@Secured** annotation should be enabled.|

- @RolesAllowed("ADMIN") is a JSR-250 annotation. 
- Test had a question suggesting @RolesAllowed could be disabled by prePostEnabled.
- @PreFilter and @PostFilter are used for filering method arguments (@PreFilter) and return values (@PostFilter) when those are collections.
- They use SpEL with some auto iteration, you can for example check if the method user has the right authorization for each returned document in a collection.

### View Resolution - setup

- Spring Boot automatically configures a ViewResolver when you include a templating engine on the classpath.
- If you do not include template engine as dependency, no view resolver is configured
- Thymeleaf is not automatically included in spring-boot-starter-web
- Include Thymeleaf like this: `spring-boot-starter-thymeleaf`
- Without Spring Boot, you must register a ViewResolver if you want to return HTML views, even if you have a templating engine on classpath
- List of template engines:
    - Thymeleaf
    - Mustache
    - FreeMarker
    - Groovy Templates

### View Resolution - use

- You store html template files in like: `src/main/resources/templates/home.html`
- In controller method, work on the `Model` object: `model.addAttribute("message", "Welcome to Spring MVC!");`
- The 'ModelAndView' type object is a bit outdated (was mentioned in some question)
- the `return "home";` statement returns the home.html content with model parameters inserted on placeholders
- Return type of controller method is String
- Thymeleaf is a template engine
- Spring Boot auto-configures a ThymeleafViewResolver when you add Thymeleaf to your classpath
- A ViewResolver maps the name "home" to a template file (src/main/resources/templates/home.html)
- _**They can map a String (the logical name of the view) to a View.**_ (was in test)
- And it tells Spring which template engine to use

### View Resolution - "Model" object

- `Model` is not a bean but an interface. Is created per request.
- It is injected as argument in controller method
- When Spring notices, it creates and populates a Model object behind the scenes, typically a BindingAwareModelMap instance.
- This model is then passed to the view rendering phase (e.g., Thymeleaf).

### Actuator Health Indicators

Spring Boot autoconfigures the following, if available:

- **CassandraDriver**HealthIndicator: Checks that a Cassandra database is up.
- **Couchbase**HealthIndicator: Checks that a Couchbase cluster is up.
- **DataSource**HealthIndicator: Checks that a connection to DataSource can be obtained.
- **DiskSpace**HealthIndicator: Checks for low disk space.
- **ElasticsearchRest**HealthIndicator: Checks that an Elasticsearch cluster is up.
- **Hazelcast**HealthIndicator: Checks that a Hazelcast server is up.
- **InfluxDb**HealthIndicator: Checks that an InfluxDB server is up.
- **Jms**HealthIndicator: Checks that a JMS broker is up.
- **Ldap**HealthIndicator: Checks that an LDAP server is up.
- **Mail**HealthIndicator: Checks that a mail server is up.
- **Mongo**HealthIndicator: Checks that a Mongo database is up.
- **Neo4j**HealthIndicator: Checks that a Neo4j database is up.
- **Ping**HealthIndicator: Always responds with UP.
- **Rabbit**HealthIndicator: Checks that a Rabbit server is up.
- **Redis**HealthIndicator: Checks that a Redis server is up.
- **Solr**HealthIndicator: Checks that a Solr server is up.

### @ControllerAdvice

- @ControllerAdvice annotated classes are used to handle cross-cutting concerns for controller classes
- Instead of writing @ExceptionHandler methods in every controller, you can define them once in a @ControllerAdvice class. This reduces duplication and ensures consistent error responses.
- @ControllerAdvice annotated classes are configured by slice test classes (@WebMvcTest)

### Random server port

- Set the server.port=0 property in the application.properties file of the test resources

### Mock annotations

| Annotation               | Purpose / What it Mocks                                    | Typical Usage Context                          |
|--------------------------|-----------------------------------------------------------|-----------------------------------------------|
| **@MockBean**              | Creates and injects a mock bean into Spring context    | Mocking dependencies in Spring Boot tests     |
| **@SpyBean**               | Injects a Mockito spy (partial mock) into Spring context  | When you want to spy on a real bean            |
| **@InjectMocks**| Mockito annotation |             |
| **@WithMockUser**          | Mocks a Spring Security authenticated user                | Secured method or controller tests             |
| **@WithAnonymousUser**     | Simulates an anonymous (unauthenticated) user             | Testing access control for anonymous users     |
| **@WithUserDetails**       | Loads a real user from `UserDetailsService` and mocks auth| Testing with specific user details              |
| **@AutoConfigureMockMvc**  | Auto-configures `MockMvc` for testing MVC controllers      | Setting up mock HTTP request testing            |
| **@MockHttpServletRequest**| Mock HTTP servlet request                                  | Simulating HTTP request parameters              |

- @WithMockUser, @WithAnonymousUser, and @WithUserDetails are from Spring Security.

### Using .requestMatchers()

Correct examples:

- `.requestMatchers("/signup", "/about").permitAll`
- `.requestMatchers(HttpMethod.PUT, "/accounts/edit*").hasRole("ADMIN")`
- `.requestMatchers("/accounts/**").hasAnyRole("USER", "ADMIN")`
- `.anyRequest().authenticated());`

### TransactionTemplate

- Is used in Spring to simplify programmatic transaction demarcation and management.
- Class that provides extra control over transactions, more than with the @Transactional attributes
    - Full control over transaction propagation and behavior.
    - Easy to define custom rollback logic.
    - Helps when annotation-based style is too limiting.

### Scope

I was not aware of scope **Application**. 

|Scope|Description|
|----|----|
|singleton|(Default) Scopes a single bean definition to a single object instance for each Spring IoC container.|
|prototype|Scopes a single bean definition to any number of object instances.|
|request|Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.|
|session|Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.|
|application|Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.|
|websocket|Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext.|



