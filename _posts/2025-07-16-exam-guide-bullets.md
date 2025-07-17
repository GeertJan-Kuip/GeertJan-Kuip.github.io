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
- You need 

## Section 3 - Spring MVC

## Section 4 - Testing

## Section 5 - Security

## Section 6 - Spring Boot



