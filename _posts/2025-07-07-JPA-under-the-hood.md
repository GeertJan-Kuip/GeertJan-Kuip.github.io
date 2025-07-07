# JPA under the hood

I had trouble understanding Spring JPA, which might be because I had trouble understanding why you would need something to simplify 'persistance'. To improve my understanding of how and why you would use all the shortcuts Spring JPA offers I first wanted to understand a liitle more of those things hidden by Spring Boot.

## General description

JPA stands for Java Persistence API. It has its origins in Java/Jakarta, not Spring. It is meant to separate the world of programming, where entities are instances objects with fields set to specific values, from the world of databases, where entities are represented as rows in a table. 

The benefit of JPA is not only that it spares you the writing of code, but also that it decouples the application from the 'persistence layer,' so that it becomes easier to change one database type for another without having to rewrite large parts of the application. The programmer deals with the same object instances and methods, no matter the database.

## Five classes

JPA is a Java thing, not a Spring invention. Spring utilizes it and adds extra layers that make working with JPA easier. There are five classes/interfaces that play a key role in Spring JPA. 

### DataSource

DataSource is a standard Java interface from the java.sql module or the javax.sql package. It manages the connection, or rather a pool of connections, to a database and is implemented by a driver vendor. As opening and closing a database is expensive, optimization is being done.

Spring by default uses HikariCP to manage the connection pool. You can configure things in application.properties like this:

```
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=myuser
spring.datasource.password=mypassword
```

The DataSource type object, which is a bean, is passed to EntityManagerFactory and JpaTransactionManager, two classes that will be discussed as well. Every time a transaction begins in your program, a connection is borrowed from the pool. The connection is released when the transaction commits or rolls back. 

### JpaTransactionManager

This is a native Spring class, used to bridge Springs transaction system (@Transactional) with JPA. Its FQN is `org.springframework.orm.jpa.JpaTransactionManager`. It is a bean in Spring.

It is important to note that everything related to database operations is done within transactions, no matter whether an @Transaction annotation is being used or not. For every transaction an EntityManager object is created. A transaction can have one or many operations and ends with a commit. 

JpaTransactionManager is the class that organizes the lifecycle of the EntityManager object (more on that later). It does the following:

1. Starts a transaction before the method runs
2. Injects a transaction-bound EntityManager
3. On success: Calls entityManager.flush()
4. Then calls commit() on the underlying JDBC connection
5. On failure (exception thrown): Calls rollback()

To be able to create an EntityManager it must know the EntityManagerFactory. Spring Boot wires this automatically but if you create JpaTransactionManager manually you must provide it as an argument, as below:

```
@Bean
public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {

	return new JpaTransactionManager(emf);
}
```

### Repository

Spring offers a family of repository interfaces that, if you extend them in your code, will be found and registered by Spring based on the presence of the @EnableJpaRepositories annotation. This annotation is included in @SpringBootApplication. 

#### Scanning for repositories

The scanning is done by default in the package where the @SpringApplication class lives, plus al its subpackages. If you want to change that, you can use the basePackages attribute:

```
@EnableJpaRepositories(basePackages = "com.example.repositories")
```

#### Runtime proxy

It is important to note that while you define a repository as an interface in your code, Spring will create a runtime proxy of it that has the status of bean with scope Singleton. This proxy bean can be inserted with @Autowired.

#### Methods in repositories

In a repository you declare methods to interact with domain objects. Without repositories, this work would have to be done in EntityManager which would be cumbersome.

Example of a repository:

```
public interface PersonRepository extends JpaRepository<Person, Long> {

	List<Person> findByLastName(String lastName);
}
```

The method names are interpreted by Spring as queries, following some specific rules. Furthermore, each repository knows to which EntityManagerFactory it belongs because this bean is injected, and therefore it also knows to what EntityManager it is related. It also knows what domain object it has, based on the generic type parameter ('Person' in the above example). Btw 'Long' is the type of the primary key.

#### Handy subinterfaces

While 'Repository' is the marker interface that is scanned for, there are various repository interfaces that extend Repository and you can use them instead of the basic Repository. They differ in the method declarations they provide and it can be convenient to use them. Here are a few:

- CrudRepository<T, ID>
- PagingAndSortingRepository<T, ID>
- JpaRepository<T, ID>

Every next type extends the previous type, so JpaRepository has all the methods of the others. Among them are `save()`, `delete()`, `findById()`, `findAll()`, `findAll(Pageable)`, `flush()`, `saveAll()`, deleteInBatch()` and more.

### EntityManager

EntityManager comes from Jakarta EE. It is an interface and its FQN is `javax.persistence.EntityManager`. The implementation is invisibly done by the JPA provider, by default this is Hibernate.

#### Scope

It has the special property that, unlike all the others, its scope is 'transaction,' or to use the Hibernate term, 'session'. The lifecycle of an EntityManager object is controlled by the JpaTransactionManager.

#### What it does

EntityManager manages the so-called 'persistence context.' To understand what it is, it is important to remember that JPA lets programmers think in terms of objects, not in terms of tables, columns and rows. 

In the persistence context, which is a cache managed by EntityManager, you find 'managed' instances of @Entity annotated objects on which operations from the repositories have been applied. These managed entities are sort of waiting for the changes made to them to become persistent (stored in the database). 

One rule that applies is that per persistent entity (a row in the database) there can only be one entity in the persistence context. 

The EntityManager can do a flush operation, which means that the persistence context (objects) is synchronized with the database. All the operations on objects done by the application are translated into sql operations and the persistent context is cleared. 

#### EntityManager and Repositories

In a Spring Boot application, or generally when you work with Spring Data JPA, you typically do not call EntityManager methods directly (although it is perfectly fine to do so). Instead, you use the repositories, which act as a convenient inbetween layer.

### EntityManagerFactory

As EntityManagers have a lifecycle of the duration of a transaction, a factory bean exists that generates EntityManagers. The EntityManagerFactory, also from Jakarta, is generated by Spring Boot automatically but you can do it yourself if you want more control.

JpaTransactionManager has the EntityManagerFactory injected, so it can create an EntityManager if one is needed.

#### FactoryBean<T>

The thing I didn't understand at first was the way that an EntityManagerFactory was created in a configuration class. It is shown in a video tutorial of the Spring Course and has this code:

```
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    // I leave this out
}
```

It is strange that the method returns a LocalContainerEntityManagerFactoryBean object instead of a EntityManagerFactory object. This has to do with a pattern that is often used in Spring and relies on the FactoryBean interface. LocalContainerEntityManagerFactoryBean is a class that implements `FactoryBean<EntityManagerFactory>`, and Spring searches for it.

When you inject an object that implements FactoryBean, Spring will not insert the object itself but the object that is generated when its `getObject()` method is called. LocalContainerEntityManagerFactoryBean must be seen as a helper object that generates an EntityManagerFactory object. This is the source code of FactoryBean:

```
public interface FactoryBean {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```

FactoryBean is used often by Spring, not only for EntityManagerFactory. You recognize the types implementing it by their postfix FactoryBean.

### Domain objects

Domain objects are annotated with @Entity and map to a database table. In Spring Boot, the @Entity annotated classes are being scanned so the application can build a database schema. 

#### Spring annotations for domain objects

There are a lot of annotations that you can use in a domain class to affect the schema, like:

|Annotation|Function|
|----|----|
|@Entity|Marks the class as a table-mapped entity|
|@Table(name = "...")|Changes the table name and adds schema-level options|
|@Id|Marks the primary key column|
|@GeneratedValue|Specifies how the primary key is generated (e.g., AUTO, IDENTITY, SEQUENCE)|
|@Column(name = "...", nullable = ..., length = ...)|Customizes the column name, size, nullability, precision|
|@OneToOne, @OneToMany, @ManyToOne, @ManyToMany|Defines relationships between entities → affects foreign keys, join tables|

#### Hibernate annotations for domain objects

If you are using Hibernate (the default) you can use Hibernate annotations as well:

|Hibernate Annotation|Purpose|
|----|----|
|@Type|Customize the SQL type used for a field|
|@CreationTimestamp, @UpdateTimestamp|Automatically populate timestamps|
|@Formula|Define computed/virtual columns|
|@NaturalId|Define natural/business keys (not surrogate IDs)|
|@OnDelete|Add ON DELETE CASCADE constraints|
|@ColumnDefault|Specify default column values|
|@DynamicInsert, @DynamicUpdate|Optimize insert/update SQL generation|

#### Scanning for @Entity classes

If you use Spring Boot, Spring Boot will scan automatically for @Entity annotated classes in the base package and its subpackages (same procedure as for repositories). You can manually add packages to scan by using:

```
@EntityScan(basePackages = {…}) 
or 
@EntityScan(basePackageClasses = {…})
```



