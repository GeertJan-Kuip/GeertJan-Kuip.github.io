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
- [This is the recent version}(https://docs.spring.io/spring-boot/api/rest/actuator/index.html)

### Return NO_CONTENT 204

- @ResponseStatus(HttpStatus.NO_CONTENT)
- return ResponseEntity.noContent().build();
- return HttpStatus.NO_CONTENT;
- return new ResponseEntity<>(headers, HttpStatus.NO_CONTENT);
- response.setStatus(HttpServletResponse.SC_NO_CONTENT); (a bit odd)

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






