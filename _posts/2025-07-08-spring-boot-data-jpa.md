# Spring Boot Data JPA

The previous blog post went in-depth on Java JPA, discussing the underlying classes and interfaces that make JPA work. This post is just about the contents of the [video tutorials](https://spring.academy/courses/spring-boot/lessons/spring-boot-spring-data-jpa-jpa) and therefore gives more idea about what to expect on the test.

## Video 1 - Spring Data JPA

### Enabling JPA

In pom.xml you add Spring JPA in the following way:

```
<dependencies>
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
 </dependency>
</dependencies>
```

It resolves a lot of packages that together make JPA work. If JPA is found on the classpath, Spring will autoconfigure the following objects:

- DataSource
- EntityManagerFactoryBean
- JpaTransactionManager

### Using @EntityScan

By default, Spring scans for domain objects ("entities") in the package where @EnableAutoConfiguration sits plus its subpackages. This is identical to the search for repositories. If you want Spring to scan at other locations, do `@EntityScan("other.package")`

### Settings in application.properties

Many thing go automatically but there are some settings in application.properties that you might need. The following are discussed:

```
#application.properties

# With default, Hibernate will try to determine it. Recommended approach.
spring.jpa.database=default  

# How to deal with database and its schema. Default is:
#   Embedded database: create-drop
#   Non-embedded: none (do nothing)
# For production, do not set it at all (none)
# Options: validate | update | create | create-drop
spring.jpa.hibernate.ddl-auto=update 

# Show SQL being run in log, well-formatted
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format-sql=true

# Set Hibernate property 'xxx':
spring.jpa.properties.hibernate.xxx=???
```

## Video 2 - Create Repositories

### Two steps

To set it up, you need two things:

- Create domain classes and annotate them with JPA annotations like @Entity, @Id, @Table etc
- Create repositories by extending Repository interface or its children

### Annotate a domain class

Example from video (btw this is a JPA, not a Spring thing):

```
@Entity  // makes class a persistent class
@Table(..)  // use if you want to have a name for the db table other han the class name
public class Customer {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;  // key must be Long type
  private Date orderDate;
  private String email;

  // getters and setters
}
```

These annotations come from JPA and Spring utilizes them. As Spring Data works also with systems other than JPA that do not have their own annotations, like MongoDB, Neo4J or Gemfire, Spring has introduced custom annotations for them as well. They are not part of the course, but it illustrates the universal character of Spring Data.

### Choose a Repository to extend

To create a repository, extend `Repository<T, ID>` or any of its children (might even be multiple of them). No annotations needed, Spring will create a proxy bean from it that can even be injected into another bean.

Repository is a marker interface, CrudRepository has crud methods included, PagingAndSortingRepository extends CrudRepository and has methods like Iterable, Page, findAll(Sort), findAll(Pageable), and JpaRepository extends PagingAndSortingRepository.

### Sample repository

Here some valid code. Note that you never have to provide the method implementations. Also note that the names of methods follow a convention and will be translated to sql statements under the hood.

```
public interface CustomerRepository extends CrudRepository<Customer, Long> {  // Long is type of Id

   public Customer findFirstByEmail(String someEmail); 
   public List<Customer> findByOrderDateLessThan(Date someDate);
   public List<Customer> findByOrderDateBetween(Date d1, Date d2);

   Customer findFirstByEmailIgnoreCase(String email);  // case insensitive search

   @Query("SELECT c FROM Customer c WHERE c.email NOT LIKE '%@%'")
   public List<Customer> findInvalidEmails();

   @Query("select u from Customer u where u.emailAddress=?1")
   Customer findByEmail(String email);  // ?1 is replaced by method parameter.
}
```

The names of methods have elements like "findFirstBy", "findBy", "LessThan", "Between" etc. To write your own query, use @Query and use the query language of the underlying product. The example here uses JPQL. You can use placeholders for the arguments of the method. Mainly used for more complex queries that would result in very long method names.

### Find repositories

Spring Boot scans for repositories (all interfaces extending Repository or a child of it) in the package where @SpringBootApplication sits plus its subpackages.

If you want Spring Data to scan in other packages as well, do this:

```
@Configuration
@EnableJpaRepositories(basePackages="com.jan.repository")
public class CustomConfig{ ... }
```

### Injecting repository as bean

As a proxy class bean is generated of the interface, and as this bean is available from the start, you can inject it in other beans, preferably in their constructor in the @Configuration class. This doesn't require an extra annotation. Example:

```
 @Bean
 public CustomerService customerService(CustomerRepository repo){
     return new CustomerService(repo);
 }
```









