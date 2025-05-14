## External properties

When running a Spring application you might need external properties from the operating system, the JVM or configuration files. Accessing them is possible and in this blog I try to get a bit more understanding of the ways in which these externals are being found, defined, and made available to the application.

### The Environment bean

Upon initialization Spring creates a special bean of type Environment. Environment is an interface that can be implemented by different classes, the most common being StandardEnvironment. The object contains a mutable list of so-called ```PropertySource<?>``` objects, each representing a source of key-value properties.

To fill the PropertySources, Spring searches at several places

It solves ambiguity by maintaining a precedence order. The order is described [here](https://docs.spring.io/spring-boot/reference/features/external-config.html), with many things I don't know much about. Below is a reduced list with the main ones. Precedence goes from low to high:

1. Configuration data files
2. Environment variables from the operating system
3. JVM System properties
4. Command line arguments 

This means you can override variables in your application.properties (or application.yml) file via the command line or via environment variables of the OS. Configuration data files (.properties, .yml, .yaml) have the following precedence order (low to high):

1. Application properties packaged inside your jar
2. Profile-specific application properties packaged inside your jar
3. Application properties outside of your packaged jar
4. Profile-specific application properties outside of your packaged jar

In this, .properties files have precedence above .yml or .yaml files. It's best to stick to one type.

### Where the external variables are

#### Configuration files

The most convenient way to set environment variables is via the application.properties file. In a Maven Spring project it resides in src/main/resources/application.properties. The names of the properties have a notation with lower case and dots. This dot notation has benefits when using it within the Spring code. Example:

```
# Server Configuration
server.port=8080
server.servlet.context-path=/myapp

# Custom Application Properties
myapp.feature.enabled=true
myapp.default-timeout=30
```

In yml/yaml files a different notation with indentation is used. It is important to note that Spring automatically maps many of these values to internal configuration values.

#### Environment variables from the operating system

On Windows and other operating systems you can set environment variables, either temporarily for the duration of your terminal session, or permanently. In the Windows command prompt you set an environment variable temporarily with 'set' and permanently with 'setx'. In Linux and on Apple you use 'export' for temporary and if you want the variable to persist, you need to add the 'export' line to the shell's configuration file.

As Linux (and maybe windows too) does not allow to use dots in environment variables, there is a mechanism in which Spring automatically converts THIS_TYPE to this.type. This allows you to set environment variables in one style and use them within Spring in 'Spring' style.

#### JVM System properties

JVM System properties will be fetched by Spring to put in a PropertySource object. If you want to add variables during startup you can add them to the java command prefixed with -D. Example:

```
java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```` 

#### Command line arguments

A very similar method, whereby you do not use the -D prefix possible. This way of doing it has less precedence so in the theoretical case where you use this and the -D in the same command with the same value name, the -D should have precedence. I didn't try it.

```
java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

### Using external variables in Spring

There are three ways to get the value of an external variable in Spring. Under the hood they all rely on the mutable list of PropertySource objects in the Environment bean.

#### @Value

When you have either an instance field or an argument in a method, you can set ('inject') its value with the @Value annotation. Examples:

```
@Value("${my.property.city}")
private String cityName;


public MyService(@Value("${myapp.default-timeout}") int timeout) {
    this.timeout = timeout;
}


private String apiUrl;

@Value("${myapp.api.url}")
public void setApiUrl(String apiUrl) {
    this.apiUrl = apiUrl;
}
```

#### @ConfigurationProperties

Using @Value makes things a bit messy. A more concise and structured way to get more than one external value inject is to use @ConfigurationProperties. There is some boilerplate involved as this requires getters and setters (or not if the instance fields are made public). The example below is the most basic variant but you can use constructor injection as well with @ConfigurationProperties, that requires a constructor that sets all instance fields. You can use Lombok as well, I don't know what it is exactly but a quick lookup tells me that it allows you to omit setters and getters (Lombok will add them to the class file 'lazily'.

Anyhow, a basic example:

application.properties:

```
myapp.feature-enabled=true
myapp.default-timeout=30
```

Component class:

```
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {

    private boolean featureEnabled;
    private int defaultTimeout;

    // Getters and setters
    public boolean isFeatureEnabled() {
        return featureEnabled;
    }
    public void setFeatureEnabled(boolean featureEnabled) {
        this.featureEnabled = featureEnabled;
    }

    public int getDefaultTimeout() {
        return defaultTimeout;
    }
    public void setDefaultTimeout(int defaultTimeout) {
        this.defaultTimeout = defaultTimeout;
    }
}
```

Using the properties in another component:

```
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final MyAppProperties props;

    public MyService(MyAppProperties props) {
        this.props = props;
    }

    public void logConfig() {
        System.out.println("Feature enabled: " + props.isFeatureEnabled());
        System.out.println("Default timeout: " + props.getDefaultTimeout());
    }
}
```

#### Environment object

It is also possible to use the Environment object itself to get the environment variables:

```
@Autowired
private Environment environment;

environment.getProperty("my.shoecolor")
```