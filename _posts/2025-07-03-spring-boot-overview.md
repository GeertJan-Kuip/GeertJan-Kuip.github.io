# Spring exam - Spring Boot 1

This are notes related to the video tutorials belonging to module 1, Spring Boot Feature introduction. In this three videos explanation is given on how Spring Boot simplifies things by using an opiniated view on Spring. Topics are dependency management, auto-configuration, integration testing and packaging. At the end a basic application is created, using JdbcTemplate and the CommandLineRunner functional interface.

## Dependencies

The two main types of dependencies are:

- A parent dependency that defines versions. It is basically an almost empty jar with a pom.xml in it.
- The 'starter' dependencies. These contain numerous transitive dependencies so that you don't have to manage those yourself.

### Parent dependency

This artifact sets versions, not only for all possible Spring dependencies but also for external libraries like Maven, Hibernate, Jackson, Tomcat etc. Add this to the pom.xml file of your project and you can omit (alomost) all further version tags:

```
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.7.5</version>
</parent>
```

### Starters

Instead of adding all Spring artifacts separately you can get them on the classpath in groups. There are a bunch of artifacts that bundle lots of other Spring artifacts which saves work and reduces complexity. These artifacts have 'starter' in their names. Add them like this:

```
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>  // resolves ~ 18 jars
    <!-- no version needed-->
  </dependency>
</dependencies>
```

Other 'starters':

```
spring-boot-starter-test
spring-boot-starter-jdbc
spring-boot-starter-data-jpa
spring-boot-starter-web
spring-boot-starter-batch
```

A list with all starters is here: https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter . Spring Initializer also helps you with this, when selecting options it will create a pom file with the right starters.

## Auto-Configuration

Auto-configuration is made possible by the @EnableAutoConfiguration annotation on the entry class. What you can basically do is merge the application class (containing the main method) and the @Configuration class. In Spring Boot this is one.

### Basic code

```
@SpringBootConfiguration  // simply extends @Configuration
@ComponentScan
@EnableAutoConfiguration
public class Application {
	
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

The code above does the following:

- Creates an appropriate ApplicationContext
- Loads all your beans
- Wires everything together
- Starts the context, plus the embedded webserver if using web

### ApplicationContext

SpringApplication is actually a Spring Boot class. The run() method returns the ApplicationContext that is created. If you want to have a reference to the applicationcontext you can do this but it is not common to do so:

```
ConfigurableApplicationContext context = SpringApplication.run(MyApp.class, args);
```

I asked ChatGPT what the explicit type of ApplicationContext is, and it said that in a typical webapplication it is often `AnnotationConfigServletWebServerApplicationContext`. Nevertheless, it can be a different one as Spring decides which one is the most appropriate.

### The @SpringBootApplication annotation

In the example above we used three annotations for the class that both acts as the entry point and the configuartion class. These three annotations can be replaced by a single annotation, namely `@SpringBootApplication`.

This does not mean that the three replaced annotations have no use at all. @SpringBootConfiguration can have a role when setting up test classes.

As an attribute to the @SpringBootApplication annotation you can use 'scanBasePackages'. This tells component scanning that it must only scan in packages that start with this elements. Example:

```
@SpringBootApplication(scanBasePackages = "com.geertjankuip.sports")
```

If you use the three annotation @SpringBootConfiguration, @ComponentScan and @EnableAutoConfiguration you need to provide this argument to @ComponentScan for the same result: `@ComponentScan("com.geertjankuip.sports")`.

### How Auto-configuration works

Spring checks what is in the classpath and in the application context to see what needs to be configured. Examples being given are:

#### Configure a DataSource

If Spring sees a dependency in the pom file of an embedded database, it will check if HSQLDB, H2 or Derby are on the classpath. If so, Spring will automatically configure a DataSource.

If Spring sees spring-boot-starter-jdbc in the pom file, and also sees that a DataSource is configured, it will configure a JdbcTemplate. If no configured DataSource is available, it will throw a runtime error: JDBC found with no DataSource.

## Packaging

Normally you need the Maven shade plugin to create a fat jar. With Spring, you can do it differently. If you add the spring-boot-maven-plugin to the build plugins, Maven will create both a fat and a skinny jar upon the command `mvn package`. The fat jar can be executed with a basic `java -jar` command. This is the pom:

```
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

This spring-boot-maven-plugin also adds a `spring-boot:run` goal to your application, which will run the application without packaging it. This is probably very convenient. Use it by running `mvn spring-boot:run`. 

## Integration testing

In Spring Boot you do not use the @SpringJUnitConfig annotation on a test class but @SpringBootTest. As attribute you provide the main class of the application, which automatically results in the loading of the configuration used for the application. You thus have the same context for the test. Example:

```
@SpringBootTest(classes=Application.class)
public class TransferServiceTests{

	@Autowired
	private TransferService transferService;

	@Test
	public void successfulTransfer(){
		TransferConfirmation conf = transferService.transfer(...);
	}
}
```

If you do not provide the main class as attribute to @SpringBootTest, SpringBootTest will search for a class annotated with @SpringBootConfiguration (or @SpringBootApplication, as this annotation includes @SpringBootConfiguration) automatically. There is only one @SpringBootConfiguration allowed in a hierarchy, and the configuration file must be in a package **above** the test. This means that the test class must reside somewhere deeper in the directory hierarchy, and thus not in a separate branch of the directory tree. _The scan moves up, not sideways._


## A simple application

You need only three files for a simple Spring Boot application, namely:

- pom.xml (assuming you use Maven)
- application.properties
- Application.java (or any other name)

Best way to start is [http://start.spring.io](http://start.spring.io). You will get everything you need, including pom file. The application.properties file will be empty.

### application.properties

The tutorial provided a simple application.properties file, namely this one:

```
# Set the log level for all modules to 'ERROR'
logging.level.root=ERROR

# Tell Spring JDBC Embedded DB Factory where to obtain DDM and DML files
spring.sql.init.schema-locations=classpath:rewards/schema.sql
spring.sql.init.data-locations=classpath:rewards/data.sql
```

### Entry class

This is a simple entry class. Note that the JdbcTemplate bean is automatically configured through auto-configuration.

```
@SpringBootApplication
public class Application {

	public static final String QUERY = "SELECT count(*) FROM T_ACCOUNT";

	public static void main(String[] args){
		SpringApplication.run(Application.class, args);
	}

	@Bean
	CommanLineRunner commandLineRunner(JdbcTemplate jdbcTemplate){
		return args -> System.out.println("there are "
			+ jdbcTemplate.queryForObject(QUERY, Long.class)
			+ " accounts");
	}
}
```

### CommandLineRunner

The thing I didn't understand here was the CommandLineRunner type bean. ChatGPT told me that there are two Spring functional interfaces, CommandLineRunner and ApplicationRunner, that are sought for automatically by Spring after the ApplicationContext has been generated. _Once they are found, their methods are executed._ The class responsible for this search-and-execute is the SpringApplication class. There is code in it like this:

```
// Inside SpringApplication
callRunners(ApplicationContext context, ApplicationArguments args) {
    // get all CommandLineRunner beans
    for (CommandLineRunner runner : getBeansOfType(CommandLineRunner.class)) {
        runner.run(args);
    }
}
```

The reason that you can create a bean of type CommandLineRunner by returning a lambda expression is that creating a lambda expression actually means creating an instance based on an interface, using the peculiarities of functional interfaces.

The use of CommandLineRunner is that it lets you run initializing code in a Spring application. You might definitely call it a [hook](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-25-hooks-listeners-and-the-observer-pattern.md).


