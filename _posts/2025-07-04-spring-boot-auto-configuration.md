# Spring Boot Auto-Configuration

The benefit of using Spring Boot over Spring Framework is that in Spring Boot a lot of bean configuration work is done automatically, based on what's on the classpath, existing beans, and property settings. This will be the topic of this blog post. Other benefits are the starters, which makes the pom files much more manageable, the use of application.properties / application.yml, the embedded server which makes it so easy to run a web application, and stuff like Actuator, DevTools Metrics etc. 

To make AutoConfiguration work, Spring uses:

- @EnableAutoConfiguration (usually via @SpringBootApplication)
- META-INF/spring.factories or spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
- Conditions like @ConditionalOnClass, @ConditionalOnProperty to selectively activate configuration

## (Auto)Configuration classes

Spring Boot has a large number of @Configuration annotated classes. They live in the Spring Boot jars, not all in the same. Spring finds them by using a list that can be found here:

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

-- or before Spring 3.x:

META-INF/spring.factories
```

### From 2.x to 3.x

AutoConfiguration is a misleading term as it suggests that it requires @AutoConfiguaration annotated classes. That is not true. Most classes used in autoconfiguration carry @Configuration, plus one or more @Conditional annotations. Spring has classes annotated with @AutoConfiguaration and they act as a sort of higher level abstraction, containing static factory methods that create beans that are @Configuration classes. But default autoconfiguration could be done without them.

Note: ChatGPT told me that @AutoConfiguration is a newer construct, before 3.x there was only @Configuration. The new list spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports has only 12 lines, all @AutoConfiguration classes. It is some sort of entry point that connects to a larger set of bot @Configuration and @AutoConfiguration files. A reason ChatGPT gave for this change was that in the old versions, the loading of the autoconfiguration classes was rather time-consuming. It is still not completely clear to me but for now it is enough.

@AutoConfiguaration becomes most relevant when you start to create your own @AutoConfiguaration class(es), these enable you to override default autoconfiguration. 

### Example

Below an @AutoConfiguration class (body excluded). It shows conditions and imports. Those imports seem to be @Configuration classes.

```
@AutoConfiguration(before = DataSourceInitializationAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceCheckpointRestoreConfiguration.class })
public class DataSourceAutoConfiguration { .. }
```

## Overriding the default configuration

You can override Spring Boot's default configuration in the following ways:

- Set some of Spring Boot's properties
- Explicitly define beans yourself so Spring Boot won't
- Explicitly disable some autoconfiguration
- Change dependencies or their versions

### Set Spring Boot's own properties

Spring Boot properties have the form `spring.{topic}.{specific1}.({specific1})`. Examples are:

```
spring.datasource.url
spring.datasource.username
spring.datasource.password
spring.datasource.driver-class-name

spring.sql.init.schema-locations
spring.sql.init.data-locations

logging.level.org.springframework // logging.level is understood by Spring, org.springframework is the way to point to a package
logging.level.com.acme.your.code  // refers to logging level of packagae com.acme.your.code
```

You can give these properties, whose names will be understood by Spring Boot, your own specific values. There are hundreds of Spring properties like these that you can utilize.

It is recommended to define these properties in the application.properties file, because this file is read early. Properties set in here, instead of in custom property files, are immediately available which might be advantageous. For example, setting the log level must be done at the very beginning of the lifecycle.

Properties in other files are usually loaded with the @PropertySource annotation, this is more appropriate for non-Spring Boot properties that you introduce yourself.

#### Connection pool settings

At the end of the video ('Override default configuration, module 2), attention is given to the configuration of the connection pool of a database. This is typically done in application.properties with these Spring properties:

```
spring.datasource.initial-size = 
spring.datasource.max-active = 
spring.datasource.max-idle =
spring.datasource.min-idle =
```

### Define your own Beans

AutoConfiguration uses conditions and one of the typical conditions is the availability of a specific bean. If the bean is already available, Spring Boot will not autoconfigure a second.

This means that if you create a bean yourself in the software, you override the bean that Spring Boot would have created. Example are the dataSource bean and the JdbcTemplate bean. These are typically beans that Spring Boot will configure if it is not already there, so creating them yourself has the effect of overriding the default Spring Boot behaviour.

Like much in Spring, autoconfiguration is generally based on bean type, not on bean name. So it is not important what name you give the bean, it is the type that matters. 

### Explicitly disable some autoconfiguration

You can disable specific autoconfiguration classes, and this can be done via annotations or via configuration, in the application.properties file.

#### Disable via annotation

You can add an attribute 'exclude' to either @EnableAutoConfiguration or to @SpringBootApplication. Example:

```
@EnableAutoConfiguration(exclude=DataSourceAutoConfiguration.class)

or

@SpringBootApplication(exclude=DataSourceAutoConfiguration.class)
```

#### Disable via configuration

In application.properties, use the spring.autoconfigure.exclude property:

```
spring.autoconfigure.exclude = org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### Change dependency or version

You might want to change a dependency or its version, for example because the default version has a bug, because of compliance or because of company policies. 

With Spring Boot you have a parent pom.xml file that defines all the versions of the dependencies being used, including the transitive dependencies and including all third party libraries (Jackson, Hibernate etc.).

To override these default versions, you must adjust the project pom file (not the parent file). You can add the following property in your pom file to override the Spring Framework version being used:

```
<properties>
  <spring-framework.version>5.3.22</spring-framework.version>
</properties>
```

It is also possible to override the contents of the typical starter packages. The example below shows how to change the standard Tomcat server library for a Jetty library:

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```





 

