# Spring Boot properties

LALA

This chapter is based on the [first video tutorial of module 2 of the SpringBoot course](https://spring.academy/courses/spring-boot/lessons/spring-boot-closer-look-look). The topic is the use of external properties in the application.properties (or application.yml) file. The precedence order of property sources and the use of @ConfigurationProperties are discussed.

## application.properties

This file will be searched for and the content will be loaded in the environment bean, so it is available within the application.

### autodetect and autoload

You can put any property in application.properties. Spring Boot will look automatically in the following places, in the order as they are listed:

- in the config subdirectory of the working directory (`file:./config/`)
- in the working directory itself `file:./`
- in the `config` package in the classpath (`src/main/resources/config`)
- in the classpath root (`src/main/resources`)

I asked ChatGPT why the first two options exist, it seems illogical to put a file meant to be part of a project not on the classpath. This was the answer:

- If you run java -jar myapp.jar in a folder that contains application.properties, Spring Boot will load it.
- This allows you to override packaged config files without modifying the JAR file.
- Useful for deploying the same JAR to multiple environments (dev/test/prod) with different external properties.

Bottom line: it allows for external configuration when you run the application. If I'm gonna build a project, I will use the classpath root as my default.

### Profile-specific property files

Spring Boot will look for profile-specific property files. If a file is in the right directory (see above) and has this name pattern, it will be loaded and connected to the specific profile if this profile is active:

```
application-{profile}.properties
```

Examples: application-local.properties, application-cloud.properties etc. Note: the application.properties file will be loaded whatever profile is active. The profile-specific files are additions/overrides.

## application.yml

In Spring Boot you can use application.yml instead of application.properties. It will be searched for in the same locations. Note: In Spring framework the .yml filetype is not accepted, and neither can you use .yml as attribute for the @PropertySource annotation.

In a yml file you need to use two spaces for indentation instead of tab. Yaml, like JSON, has the benefit of not having to repeat all prefixes. Furthermore, instead of creating additional property files for the different profiles, you can put different profiles in the same file:

```
spring.datasource:  // always loaded
  driver: org.postgresql.Driver
  username: transfer-app

--- // indicates separate logical file
spring:
  profiles: local
  datasource:
    url: jdbc postgresql://localhost/xfer
    password: secret45

--- 
spring:
  profiles: cloud
  datasource:
    url: jdbc:postgresql://prod/xfer
    password: secret45
```

Somehow the `---` separation in combination with the spring.profiles prefix value does the trick.

## Precedence

This is the precedence order of property sources (highest on top):

1. Devtools settings
2. @TestPropertySource and @SpringBootTest properties
3. Command line arguments
4. SPRING_APPLICATION_JSON (inline JSON properties)
5. ServletConfig / ServletContext parameters
6. JNDI attributes from java:comp/env
7. Java System properties
8. OS environment variables
9. Profile specific application properties
10. Application properties / YAML
11. @PropertySource files
12. SpringApplication.setDefaultProperties

I learned about 4, SPRING_APPLICATION_JSON. This is an environment variable of Json type that you set for the current session via the command line. Spring will search for it and if found will treat it as a property file. This way you can supply multiple properties when starting the application. It is not super practical I would say but it gives you options. Using -D does about the same.

## Using @ConfigurationProperties

@ConfigurationProperties lets you map the values in a properties file, or better, only those properties with a prefix you specify yourself, onto fields in a class. The names of the properties and fields must match and then Spring Boot knows what to do.

It is an efficient alternative for @Value, as you have to use the annotation only once. With @Value you must also rewrite the prefix multiple times, with @ConfigurationProperties only once.

### Simple eaxample

This is a typical example:

```
# application.properties
rewards.client.host=192.168.1.42
rewards.client.port=8080
rewards.client.logdir=/logs
rewards.client.timeout=2000
```

```
@ConfigurationProperties(prefix="rewards.client")
public class ConnectionSettings {

	private String host;
	private int port;
	private String logdir;
	private int timeout;
	..   // getters/setters
}
```

### Making properties available

There are three ways to make the properties you loaded into the class ('ConnectionSettings') available to other classes:

- @EnableConfigurationProperties on the application class (main entry)
- @ConfigurationPropertiesScan on the application class (Spring Boot 2.2.0+)
- @Component on the properties class itself

Examples:

```
@SpringBootApplication
@EnableCOnfigurationProperties(ConnectionSettings.class)
public class RewardsApplication{...}
```
```
@SpringBootApplication
@ConfigurationPropertiesScan
public class RewardsApplication{...}
```
```
@Component
@ConfigurationProperties(prefix="rewards.client")
public class ConnectionSettings{...}
```

All these three methods reult in the class annotated with @ConfigurationProperties becoming a bean.

### Relaxed binding on @ConfigurationProperties

There are some different conventions about naming styles of properties, for example envoronment variables using upper case with underscores (JAVA_HOME) and other proerty source using for example camel case. 

To ease up things, @ConfigurationProperties utilizes relaxed binding, which means that property names will be recognized even if they are written in another naming style. Example:

```
@ConfigurationProperties("rewards.client-connection")
public class ConnectionSettings{
	private String hostUrl;
	...
}
```

Any of the following values in a property file will be mapped on hostUrl:

- rewards.clientConnection.hostUrl
- rewards.client-connection.host-url
- rewards.client_connection.host_url
- REWARDS_CLIENTCONNECTION_HOSTURL

This relaxed binding makes it easier to overwrite propertiesfrom different sources, like Systemproperties, Windows environment variables etc.

