# Exam guide bullets

Next week I have planned to do the Spring Professional Develop (2V0-72.22) exam. I did some mock tests on Udemy and had a max score of 65%, which is not enough yet. Some of the questions on Udemy seemed to be a bit outdated, when I asked ChatGPT how to prepare it stressed to focus on the [exam guide](https://docs.broadcom.com/doc/vmw-spring-professional-develop-exam-guide). 

Here I will recreate the exam guide and provide bulletpoints for each topic mentioned. One of the difficult things is the sheer amount of concepts, classes and methods, making it hard to generate some overall picture in which every learnable element has its own distinct place. Hope this approach works.

## Section 1 - Spring Core

## Section 2 - Data Management

### 2.1 - Introduction to Spring JDBC

#### 2.11 - Use and configure Spring's JdbcTemplate

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

#### 2.12 - Execute queries using callbacks to handle result sets

- `.queryForObject(String query, Class<T> returntype)` returns one object.
- .queryForObject(..) can include bind variables as varargs: `.queryForObject(String query, Class<T> returntype, var1, var2, ..)`

#### 2.13 - Handle data access exceptions

### 2.2 - Transaction Management with Spring

#### 2.2.1 - Describe and use Spring Transaction Management

#### 2.2.2 - Configure Transaction Propagation

#### 2.2.3 - Setup Rollback rules

#### 2.2.4 - Use Transactions in Tests

### 2.3 - Spring Boot and Spring Data for Backing Stores

#### 2.3.1 - Implement a Spring JPA application using Spring Boot

#### 2.3.2 - Create Spring Data Repositories for JPA

## Section 3 - Spring MVC

## Section 4 - Testing

## Section 5 - Security

## Section 6 - Spring Boot



