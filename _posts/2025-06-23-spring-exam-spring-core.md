# Spring exam - Spring Core

Past month I did some other things (PowerShell scripting, Maven, JUnit, debugging) but now I want to proceed with Spring, getting the Spring Certified Professional certification this summer. I have the [exam guide](https://docs.broadcom.com/doc/vmw-spring-professional-develop-exam-guide) and will use it as a roadmap to get famililar enough with Spring.

The upcoming blog posts will each provide notes on one of the six sections of the exam guide. This is a first in the series and it discusses Spring Core. I might omit subsections if I think they aren't useful.

## Objective 1.1 - Introduction to Spring Framework

They might have a question about this. Few notes:

- Spring includes both the Servlet-based Spring MVC web framework and the Spring WebFlux reactive web framework.
- Spring Framework's jars allow for deployment to the module path. It uses Automatic-Module-Name manifest entries.

The latter is not obvious, the JPMS arrived only in Java 9 while Spring started in 2003. Maven, for example, does not work with the module path as far as I know.

### Design philosophy

- Provide choice at every level
- Accomodate diverse perspectives
- Maintain strong backward compatibility
- Care about API design
- High standards for code quality

The latter translates to 'meaningful, current and accurate javadoc, clean code structure and no circular dependencies between packages.

Generate a basic project on [start.spring.io](start.spring.io).

### IoC and DI

Inversion of Control and Dependency Injection are complementary. Dependency Injection means that the dependencies of a class (those objects that it works with and that have the form of member fields) are either initialzed by constructor arguments, by arguments belonging to a factory method or by properties that are set on the object instance after construction of it with a setter method. 

In all of these cases, the dependency is an argument to a method (constructor or setter) and created outside of the object instance itself. Some other part of the code must provide the dependency. The opposite of DI would be if somewhere in a method or a constructor or a initialization block a 'new DependencyType()' or a 'DependencyType.get()' method was being used.

Given that every dependency is initialized via injection, it becomes possible to apply the inversion of control principle. The construction of the dependencies takes place outside of the class instance itself and will be done on a central place: the 'IoC container' or 'Application context.'

### ApplicationContext

ApplicationContext is an interface that extends BeanFactory and is used as the reference type for the IoC container. The object type of ApplicationContext can be several, depending on the type of application you make. Some options:

|Class|Description|
|----|----|
|ClassPathXmlApplicationContext|Standalone apps or legacy systems using XML config files located on the classpath (e.g., applicationContext.xml). Simple to use, especially in early Spring projects.|
|FileSystemXmlApplicationContext|Desktop apps or services that load XML configuration from the filesystem, allowing externalized configuration without modifying the packaged app.|
|AnnotationConfigApplicationContext|Modern standalone apps or tests using Java-based config (@Configuration), annotation scanning, etc. Common in pure Java Spring applications.|
|AnnotationConfigServletWebServerApplicationContext|Spring Boot servlet-based web applications using embedded Tomcat/Jetty/Undertow. It handles servlet wiring and uses annotation-based config.|
|AnnotationConfigReactiveWebServerApplicationContext|Spring WebFlux applications using reactive, non-blocking servers (like Netty). Used when building reactive microservices in Spring Boot.|
|AnnotationConfigWebApplicationContext|Traditional servlet-based web applications (non-Spring Boot) using annotation-based config. Registered in web.xml or via WebApplicationInitializer.|
|GenericWebApplicationContext|A flexible and programmable context often used in frameworks or integration tests. Allows registration of beans programmatically (no annotations or XML required).|
|StaticWebApplicationContext|Mainly used for lightweight unit testing of Spring MVC components. It allows registration of mock beans without scanning or external config.|

From the documentation: _"The org.springframework.context.ApplicationContext interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on the components to instantiate, configure, and assemble by reading configuration metadata."_

## Objective 1.2 Java Configuration

### 1.2.1 Define Spring Beans using Java code

If you define Java beans without an xml file, there are two options:

- Annotation-based configuration. This requires the use of annotations like @Autowired and @Component on the application's component classes
- Java-based configuration. A class annotated with @Configuration contains code and annotations that define the beans and the wiring. The @Bean methods act as static factory methods.

Depending on what strategy you choose for defining beans and their configuration, you choose the specific implementation of ApplicationContext. For example, xml-based configuration is served by implementations that have Xml in their name.

When they talk about defining or configuring Spring beans using Java code, they mean creating beans using the @Bean in the @Configuration class. The classes that will become beans do not require annotation then, therefore it is called Java-based configuration. Importantly, the wiring is done without annotations, purely Java based.

Fun fact from the documentation: _"You can use @Bean-annotated methods with any Spring @Component. However, they are most often used with @Configuration beans."_ Note that using @Bean outside @Configuration has negative side effects. What I understand from [the documentation](https://docs.spring.io/spring-framework/reference/core/beans/java/basic-concepts.html) is that such beans will not get a proxy class and thus cannot be properly wired.

@Configuration/@Bean can be used in combination with @Component ('Annotation-based configuration') if you use the AnnotationConfigApplicationContext implementation of ApplicationContext. This one is the go-to if you want to be able to combine both methods. To enable scanning the configuration class must have not only @Configuration but also @ComponentScan (unless you use @SpringBootApplication, which effectively combines these two annotations). @ComponentScan can have an argument that tells them in which package(s) to scan. Like `@ComponentScan(basePackages = "com.acme")`.

Instead of using @ComponentScan on the @Configuration class you can register the classes annotated with @Component or one of its equivalents programmatically, either by adding them as argument when constructing the ApplicationContext or by creating the ApplicationContext without them as argument and then use the register method. Both methods illustrated below:

```
public static void main(String[] args) {
	ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
	MyService myService = ctx.getBean(MyService.class);
	myService.doStuff();
}

public static void main(String[] args) {
	AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
	ctx.register(AppConfig.class, OtherConfig.class);
	ctx.register(AdditionalConfig.class);
	ctx.refresh();
	MyService myService = ctx.getBean(MyService.class);
	myService.doStuff();
}
```

### 1.2.2 Access Beans in the Application Context

I had some trouble finding out what exactly is meant by this title. I asked ChatGPT and, among others, gave me a list of retrieval methods:

- Retrieve by type: getBean(MyService.class)
- Retrieve by name: getBean("myService")
- Retrieve by name and type: getBean("myService", MyService.class)

### 1.2.3 Handle multiple Configuration files

#### @Import

One @Configuration class can have an @Import annotation to be used for importing another @Configuration class. See [example](https://docs.spring.io/spring-framework/reference/core/beans/java/composing-configuration-classes.html). When constructing the ApplicationContext, it now is enough to have only the configuration class as argument that has imported the other configuration class. If you do not want to use @Import to combine configuration classes, you need to name all configuration classes as argument in the constructor of the ApplicationContext, like this:

```
new AnnotationConfigApplicationContext(AppConfig.class, OtherConfig.class);
```

#### @ImportResource

Using this annotation on a configuration class lets you import all configuration info from one or more xml files. Examples:

```
@ImportResource("classpath:/beans.xml")  // single xml file

@ImportResource({"classpath:/a.xml", "classpath:/b.xml"})  // multiple xml files
```

The reverse is also possible, loading a configuration bean into an xml file to combine them:

```
<!-- beans.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">

    <context:annotation-config />
    <context:component-scan base-package="com.example" />

    <bean class="com.example.AppConfig"/> // AppConfig is a @Configuration class with beans

</beans>
```

#### Combine component scanning with Java/XML

You can do configuration in a @Configuration class but do additional component scanning as well (Java based combined with annotation based):

```
@Configuration
@ComponentScan("com.example.services")
public class AppConfig {}
```

Or you add a line to the xml configuration file to activate additional component scanning:

```
<context:component-scan base-package="com.example.services"/>
```




### 1.2.4 Handle Dependencies between Beans

#### Xml-based

When using xml based configuration, you create the wiring in the xml file itself. If you have a bean ThingOne depending on beans ThingTwo and ThingThree, you create the ThingOne class as usual with a constructor with two arguments:

```
package x.y;

public class ThingOne {

	public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
		// ...
	}
}
```

In the xml file you specify the beans and the wiring. Soring understands the order in which you provide the arguments to the constructor.

```
<beans>
	<bean id="beanOne" class="x.y.ThingOne">
		<constructor-arg ref="beanTwo"/>
		<constructor-arg ref="beanThree"/>
	</bean>

	<bean id="beanTwo" class="x.y.ThingTwo"/>

	<bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

There are more variants, you can also use setter injection (Spring advises to do this only with optional dependencies). 

#### Annotation-based

You can add the following tag to your xml if you want to mix xml-based configuration with annotation-based configuration, more specifically with respect to dependency configuration:

```
<context:annotation-config/>
```

This tag activates Spring's annotation-driven dependency injection, which includes:

- @Autowired
- @Value
- @PostConstruct / @PreDestroy
- @Resource (JSR-250)
- @Inject (JSR-330)

Those annotations will be processed, whether or not there is component scanning. It gives you a combi where xml defines the beans and the annotations do the wiring. If componentscanning is enabled, and/or if beans are defined in the code itself, you do not need this tag. Note: you can torn on component scanning on in the xml file using this one:

```
<context:component-scan base-package="com.example"/>
```

Having this tag means you can do without the ```<context:annotation-config/>```.

The @Autowired annotation can be used on constructors, setters and fields. If a class has just one constructor you can omit @Autowired, since Spring version 4.3. If multiple constructors exist you need @Autowired to tell Spring which one to use.

A very similar annotation is @Inject. This one is not an origial Spring annotation (@Resource isn't either) but can be used, although using @Autowired is recommended. The difference is that @Autowired has the very convenient option that if the class to autowire doesn't exist, it injects null and does not throw an exception. Important: this only works if you write ```@Autowired(required = false)```, the default is true. @Inject will fail if the dependency doesn't exist. It doesn't have this 'required' option and will always throw something like ```NoSuchBeanDefinitionException```.

@Resource can do similar things as @Autowired but whereas @Autowired injects beans by type, @Resource injects beans by name. @Resource is used for fields and setters, not for constructors. @Autowired does all three of them. @Autowired knows what to inject based on the type (unless multiple beans with that type exist, then there are some rules for resolving this, there is for example the @Qualifier annotation that helps here). @Resource has a name as argument and resolves by name instead of type. If no argument provided, it tries to resolve by type.

@Value differs from both as it is not for injecting beans but for injecting values. It can inject the following value types:

- Property values (application.properties)
- Constants (@Value("Hello"))
- Spring Expression Language (@Value("#{2 * 60}"))

These three can be combined. I asked ChatGPT for an example and got this:

```
# application.properties
app.user.name=geert
app.timeout.seconds=15
```

```
@Component
public class WelcomeService {

    @Value("#{ 'Welcome ' + '${app.user.name}' + '! Timeout is ' + (${app.timeout.seconds:10} * 2) + ' seconds.' }")
    private String welcomeMessage;

    public void printWelcome() {
        System.out.println(welcomeMessage);
    }
}
```

Note: the `app.timeout.seconds:10` picks up an external property and gives is a default value of 10. That value would be used if the value is not set or if the value is no part of the properties at all. Spring interpretes it as: 

_"Look for the property app.timeout.seconds. If it exists, use its value. If it does not exist, use the default value 10."_

#### Java-based

In the configuration class you can inject a bean as a dependency in another bean in the following way:

```
@Bean
public BeanOne beanOne(BeanTwo beanTwo) {
    return new BeanOne(beanTwo);
}

@Bean
public BeanTwo beanTwo() {
    return new BeanTwo();
}
```

It is possible as well to do it like this, but this is not recommended (ChatGPT states that the previous method is the cleaner, safer, and more flexible approach):

```
@Bean
public BeanOne beanOne() {
    return new BeanOne(BeanTwo());
}
```

This latter method suggests that some BeanTwo instance is created, but actually it was already created and is simply retrieved from the collection that holds all created bean instances that have scope singleton.


### 1.2.5 Explain and define Bean Scopes

There are two main scopes for beans plus four others that are only relevant in web-related applications. The two main scopes are:

- **Singleton** - Only one instance of the bean is created and it lasts as long as the application runs.
- **Prototype** - A new instance is created every time a request for that bean is being made.

The four other scopes are `request`, `session`, `application` and `websocket`. They are only available if you use a web-aware ApllicationContext implementation, like `XmlWebApplicationContext`. An exception is raised if you try to use these scopes in a non-web context.

For beans, singleton is the default scope. You need @Scope("Prototype") to set it to Prototype.

## Objective 1.3 Properties and Profiles

### 1.3.1 Use External Properties to control Configuration

I have written extensively on it [here](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-05-14-external-properties.md). Basically there are three ways to inject external properties into your beans:

- using the @Value annotation
- through Spring’s Environment abstraction
- through @ConfigurationProperties.

What you can do with @Value can also be done using the Environment bean but @Value is often more convenient.

#### Using @PropertySource

You can also add @PropertySource to the @Configuration class. Thiss adds an extra file/resource with properties, above those that are picked up by Environment (Java System properties and Environment variables). For the argument in @PropertySource, you can use three types of prefixes:

- classpath:
- file:
- http: 

Examples:

```
@PropertySource("classpath:/resources/config/app.properties")
@PropertySource("file:config/local.properties")
```

The properties can be used using SpEL ("${myproperties.dbpassword}").

### 1.3.2 Demonstrate the purpose of Profiles

Profiles are groups of Beans. Some Beans might belong to no group, and thus to any, others might belong to a specific profile/group. Profiles are not mutually exclusive, you can activate more than one profile at the same time.

#### How to define a profile?

- 1: Use @Profile annotation, with profile name as argument, on @Configuration class. Everything in this class now belongs to this profile.
- 2: Use @Profile annotation, with profile name as argument, on @Bean method. The bean constructed by this method belongs to the specific profile.
- 3: Like (1), but make sure that you use @Profile("X") on one @Configuration class and @Profile("!X") on another (mutually exclusive).


(1) Lets you do a lot at once, you might think of creating one @Configuration class for every profile. (2) is better if the profiles are rather similar. You can even create two @Bean("MyName") variants (they have the same name) but give them a profile that is the inverse of the other. Example:

```
@Bean(name="DataSource")
@Profile("embedded")
public DataSource dataSourceForDev(){
	//
}

@Bean(name="DataSource")
@Profile("!embedded")
public DataSource dataSourceForProd(){
	// You might create a different explicit type here, note that both methods return the same interface.
}
```

Using the exclamation mark (!) is a good technique, because it guarantees that only one of two beans will be used.

#### Activating a profile(s)

1- Using the command line

```
-Dspring.profiles.active=embedded,production
```

2 - Programmatically:

```
System.setProperty("spring.profiles.active", "embedded,production"); // must be done before creating ApplicationContext
SpringApplication.run(AppConfig.class);
```

3 - Integration Test _only_: @ActiveProfiles (not covered here)

#### Using Profile to control @PropertySource

The sample below shows that you can decide which property file is used based on profile:

```
// When "local" profile, "local.properties" file
@Configuration
@Profile("local")
@PropertySource("local.properties")
Class DevConfig {}

// When "cloud" profile, "cloud.properties" file
@Configuration
@Profile("cloud")
@PropertySource("cloud.properties")
Class ProdConfig {}
```

### 1.3.3 Use the Spring Expression Language (SpEL)

The Spring Expression Language is quite extensive. The basic usage is something like this:

```
@Value("#{ systemProperties['user.region'] }")
```

In a SpEL expression you can use the names of beans, those will be recognized. For example:

```
@Value("#{strategyBean.keyGenerator}") 
```

This evaluates to the keyGenerator field that lives in strategyBean.

Note that properties in @Value annotations can be accessed in two ways, either as a property using ${} or as a SpEL expression using #{}. The following two are equivalents:

```
@Value("${daily.limit}")
@Value("#{environment['daily.limit']}")
```

Note that in the second case, daily.limit must have quotation marks as system- and environment variables (and properties in general) are always strings. The first case, ${daily.limit}, returns a string for the same reason. If you want to make calculations with properties, you must cast the string to a number first:

```
@Value("#{new Integer(environment['daily.limit']) * 2}")  // OK
@Value("${daily.limit * 2}")  // NOT OK, requires cast
```

#### Fallback values

Both ${} and #{} have a way to provide a default/fallback value:

```
@Value("${daily.limit : 1000}")   // just :
@Value("#{environment['daily.limit'] ?: 1000}")  // Elvis operator
```


## Objective 1.4 Annotation-Based Configuration and Component Scanning

### 1.4.1 Explain and use Annotation-based Configuration

Annotation-based configuration means that beans are being found by scanning. The stereotype annotations mark the classes that will be beans. 

#### Using @Autowired

Dependency injection is done with @Autowired. This annotation makes sure that the objects(s) you specify as argument(s) to a constructor or method will be the bean instances of those classes. Field injection is not recommended but can be done with @Autowired. The problem with field injection is that it makes testing more difficult (during unit tests, the Spring framework isn't available to do the injection).

When there are multiple constructors in a Bean class, you put the @Autowired annotation at the constructor that must be selected from those. When there is only one constructor you can omit the @Autowired annotation.

@Autowired throws an exception if the required dependency is not found. You can suppress this exception-throwing by providing the attribute 'required', like `@Autowired(required=false)`. An alternative to using the 'required' attribute is to wrap the bean in an optional:

```
@Autowired
public void setAccountService(Optional<AccountService> accountService){
	this.accountService = accountService;
}
```

#### Using @Qualifier

@Autowired finds dependencies by type. It is possible that there are two beans that implement the same interface and that the @Autowired constructor or method has that interface type as argument. This results in exception as Spring doesn't know which one to choose. To solve it, give both beans a different name using their stereotype annotation (@Component("someName")) and add a @Qualifier annotation directly in front is the ambiguous argument:

```
@Autowired
public TransferServiceImpl( @Qualifier("someName") AccountRepository accountRepository){
	// code
}
```

So the @Qualifier helps to solve ambiguity about which bean to use for injection.

#### Combining @Autowired and @Value

if you want to use @Value on one of the arguments of method or constructor, you must add the @Autowired annotation on the method or constructor itself. This surprised me as I didn't regard the injection with a value instead of a bean as injection. Example of right syntax for constructor or method:

```
@Autowired // optional if this is the only constructor
public TransferServiceImpl(@Value("${daily.limit}") int max){
	// code
}

@Autowired
public void setDailyLimit(@Value("${daily.limit}") int max){
	// code
}
```

Instead of using ${} I could also have used SpEL (#{}).

#### Using @Lazy

Adding @Lazy to a component results in lazy initialization of that specific bean. Useful if bean's dependencies are not available at startup. Note: in Java-based configuration you can use @Lazy as annotation to the bean constructing methods in @Configuration.


### 1.4.2 Discuss Best Practices for Configuration choices

When using annotation-based configuration, component scanning is done, by default on all files on the classpath. This might take long. You can limit the scanning by providing one or more arguments to @ComponentScan. Good practice is to make the number of files to scan for components as small as possible. Do not scan in libraries you import, it takes very long time.

Good example: `@ComponentScan({"com.geertjankuip.fish", "com.geertjankuip.boat"})`  
Bad example: `@ComponentScan("com")`  // allows too much

Often you will mix-up annotation-based and Java-based. Java-based can do one thing that annotation-based can't namely creating beans from class definitions in libraries. Generally, the @Bean constructing methods provide a powerful way to separate business form beans, and it allows you to use legacy classes without having to modify them.

### 1.4.3 Use @PostConstruct and @PreDestroy

In line with the three ways to define and configure beans, there are three ways to controll bean lifecycle behaviour. The most modern way is to add methods to the bean classes that are annotated with either `@PostConstruct` (initializing a bean) or `@PreDestroy` (destroy a bean). The `PostConstruct` method is called right after the bean has received all of its dependencies.

A less recommended but valid way is to add the InitializingBean and DisposableBean interfaces on your bean class. They force you to implement the following methods:

```
void afterPropertiesSet() throws Exception;
#and
void destroy() throws Exception;
````

If you go pure xml, you can add 'init-method' to the xml tag of bean:

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init" destroy-method="cleanup"/>
```

Subsequently, the methods you named 'init' and 'cleanup' must be used in the bean class like this:

```
public class ExampleBean {

	public void init() {
		// do some initialization work
	}

	public void cleanup() {
		// do some cleanup work before destroying bean
	}
}
```

If you want, you can tell Spring in the xml file to look for a specific method name in every bean class that will be used as postconstruct or predestroy method. You do that by adding it to the `<beans>` tag that is the parent of the `<bean>` elements:

```
<beans default-init-method="init"> <!-- every method named init in any bean will be called after the bean is created-->

	<bean id="blogService" class="com.something.DefaultBlogService">
		<property name="blogDao" ref="blogDao" />
	</bean>

</beans> 
```

#### Most relevant for exam

For the exam, the first methods (using @PostConstruct and @PreDestroy) are the important ones. They are also the most up-to-date way of using Spring.

#### @PostConstruct and @PreDestroy must return void & have no arguments

Methods annotated with @PostConstruct or @PreDestroy cannot have return values and can take no arguments.

#### Combining with Java-based config

In Java-based configuration, you have the option to create beans outside of your own code (in libraries or somewhere in your legacy code). But then, how do you tell which methods should be called post-construct or predestroy? 

An alternative for @PostConstruct and @PreDestroy is this:

```
@Bean(initMethod="populateCache", destroyMethod="flushCache")
```

Of course, these methods must be available in the library/legacy class, otherwise they cannot be called. But the benefit here is that you can leave the class code as is and do everything in the @Configuration class.


### 1.4.4 Explain and use “Stereotype” Annotations

The most important stereotype annotations are Stereotype annotations are @Component, @Controller, @RestController, @Service, @Repository and @Configuration. They all indicate that the annotated class is a bean.

You can use these and other annotations as meta annotations to create your own custom stereotype annotation.

## Objective 1.5 Spring Bean Lifecycle

### 1.5.1 Explain the Spring Bean Lifecycle

#### Initialization steps

These are the steps:

- Load Bean definitions
- Post Process Bean definitions

Subsequently, for each bean:
- Find/create dependencies for the bean (decide order based on this)
- Instantiate Bean
- Perform Setter/Field Injection
- Bean Post Processors before initialization
- Initialization
- Bean Post Processors after initialization
- Bean ready for use


#### Destruction phase

- All beans are cleaned up
    - Any registered @PreDestroy methods are invoked
    - Beans released for the Garbage Collector to destroy
- Also happens when any bean goes out of scope
    - Except Prototype scoped beans

This only happens if the application is gracefully closed (context.close()).

### 1.5.2 Use a BeanFactoryPostProcessor and a BeanPostProcessor

A BeanFactoryPostProcessor is a bean that has interface BeanFactoryPostProcessor implemented. This gives it a method postProcessBeanFactory. This method is called by Spring on the moment in the lifecycle that it has registered all beans but not created instances of them. A BeanFactoryPostProcessor allows you to modify bean definitions. More about its internal workings in another [blog post](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2025-06-25-hooks-listeners-and-the-observer-pattern.md).

A BeanPostProcessor is run after the instances of the beans are created and dependencies are injected (actually much of this work is done by BeanPostProcessors). Its basic workings are very similar to that of BeanFactoryPostProcessor.

#### BeanFactoryPostProcessor 

Two Spring BeanFactoryPostProcessors are important, namely PropertySourcesPlaceholderConfigurer and ConfigurationClassPostProcessor.

_**PropertySourcesPlaceholderConfigurer**_

It resolves placeholders like ${some.property} in:

- @Value annotations
- `<bean>` XML definitions
- @Bean method parameters

Before beans are created, it scans through PropertySources (like application.properties) and replaces `${...}` with real values.

_**ConfigurationClassPostProcessor**_

This processor:

- Reads @Configuration classes
- Finds @Bean methods
- Handles @ComponentScan, @Import, @ImportResource, etc.
- Registers new bean definitions from those

Without this, Java config classes wouldn’t work at all.

It is important to note that in Java-based and annotation-based configuration scenario's, almost all the worki is done in th epost-processing phase, not in the loading phase that precedes it.

#### BeanPostProcessor

When bean instances are created and dependencies are injected, the BeanPostProcessors can do their work. They operate on the real bean instances, not on their definitions as the BeanFactoryPostProcessors do. BeanPostProcessors cause initialization methods to be called, such as @PostConstruct or the xml based init-method. See 1.4.3. 

Spring has its own BeanPostProcessor implementations that run by default, like CommonAnnotationBeanPostProcessor. This class enables annotations like @PostConstruct, @PreDestroy and @Resource.

Another relevant BeanPostProcessor is AutowiredAnnotationBeanPostProcessor. This processor handles the @Autowired annotations. It also handles part of the @Value annotations, although here the PropertySourcesPlaceholderConfigurer is also required to resolve the placeholders. 

BeanPostProcessor is called an 'Extension point'. It is powerful:

- Can modify bean instances in any way.
- Powerful enabling feature (? text from video tutorial)
- Will run against every bean.
- Can modify a bean before and/or after initialization.

The BeanPostProcessor interface has two default methods, `postProcessAfterInitialization()` and `postProcessAfterInitialization()`. Their names tell the difference. This is the code:

```
public interface BeanPostProcessor {

	default @Nullable Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	default @Nullable Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
```

As you see the both return the bean object that is the first argument. Within the method you can change the bean however you like and you can return anything. There aren't many restrictions. Nevertheless, this option is not used often, this phase is mainly run by Spring's own BeanPostProcessors.

About the latter: ChatGPT listed seven relevant Spring BeanPostProcessors but I'll leave that list for now. It is remarkable that here again most of the work is done in the post-processing phase, not in the phases preceding it (although that is what the schema suggest).

### 1.5.3 Explain how Spring proxies add behavior at runtime

In the BeanPostProcessing phase, the BeanPostProcessors can be written in a way that they do not return the original bean that was inserted as first argument, but a different, strongly modified bean, also called a proxy. Technically, it wraps the original bean in a wrapper with other code and returns this wrapped object. From then on, Spring will only know and return this proxy bean when you call getBean(). 

Whether a bean will be replaced by a proxy depends on the annotations it carries. Annotations that induce proxy-creation are:

- @Transactional
- @Async
- AOP (@Aspect)
- @Secured, @PreAuthorize (Spring Security)
- @Scope("request") or @Scope("session")

There are two ways for Spring to create a proxy. There is the (modern) JDK Proxy that requires an interface to be implemented in the bean. The proxy object will be based on this interface, just like the original bean. These proxies are called _dynamic proxies_. This API for the JDK proxy comes from the JDK.

The other way is the CGLib Proxy, not part of the JDK, but included in Spring jars. Used when no interface implemented, cannot be applied to final classes or methods. The technique is to create a subclass that extends the original bean. Remarkable: SpringBoot relies solely on CGLib proxies, not on JDK proxies.

### 1.5.4 Describe how Spring determines bean creation order

Beans must be created _after_ their dependencies. To achieve the right order, Spring evaluates dependencies for each bean and then follows a recursive process. It creates a bean if either the bean has no dependencies or if the dependencies are already created.

The process of analyzing dependencies takes place after BeanFactoryPostProcessors have done their work and before the beans are instantiated.

There is an annotation @DependsOn() that you can use on a class or on a @Bean annotated constructor to make explicit the dependencies. It is not required, Spring will find out anyway. I think @DependsOn(someOtherBean) forces Spring to create someOtherBean first, even if it isn't a dependency. 

Singleton beans are what is called 'eagerly' instantiated, unless marked as _Lazy_. The latter bean type will only be instantiated when the bean is required.

#### Naming beans

No chapter deals with this, so here: when creating beans using annotations, the bean name is derived from class name or from the annotation value attribute. When creating beans with java code, the name is the name of the method that creates the bean or the name of the name/value attribute of @Bean. With the latter method, whereby the bean is a return value of a method, you can give the bean an interface reference type. 

### 1.5.5 Avoid issues when Injecting beans by type

There is a scenario discussed in [the video](https://spring.academy/courses/spring-framework-essentials/lessons/spring-essentials-spring-container-bean-creation-injection-issues) where you might encounter problems. When you do Java-based configuration, you can give beans an interface type instead of an explicit type. In the wrong scenario a bean class implements multiple interfaces, say A and B, you create the bean as type A but inject using type B. Spring won't recognize what happens (and someone reading your code might neither).

Generally, using interface type is regarded good practice. Even if the return type of an @Bean method is an explicit class type, it is good practice to use the interface type as argument in the dependency injection process. It is just regular Java practice.

## Objective 1.6 Aspect Oriented Programming

### 1.6.1 Explain the concepts behind AOP and the problems that it solves

AOP solves cross-cutting concerns in one place (to 'modularize' it). Examples of such concerns are:

- Logging and Tracing
- Transaction Management
- Security
- Caching
- Error Handling
- Performance Monitoring
- Custom Business Rules

There is AspectJ and Spring AOP. The former modifies bytecode, the second creates proxy classes. There are elements of AspectJ in Spring AOP. The course will be mainly about Spring AOP.

Core AOP concepts:

- Join Point - a point in the execution of a program such as a method call or an exception thrown.
- Pointcut - an expression that selects one or more Join Points
- Advice - code to be executed at each selected Join Point
- Aspect - A module that encapsulates pointcuts and advice
- Weaving - technique by which aspects are combined with main code
- Proxy 
- AOP Proxy - an enhanced class in place of the original with Aspect behaviour woven into it

### 1.6.2 Implement and deploy Advices using Spring AOP

In your @Aspect class, the place where you define pointcuts and advice ("where and what") you can define pointcuts in a sort of generic way. The example in the video is this:

```
@Aspect
@Component
public class PropertyChangeTracker {
	private Logger logger = Logger.getLogger(getClass());

	@Before("execution(void set*(*))")
	public void trackChange(){
		logger.info("Property about to change...");
	}
}
```

Note the value of @Before. It is a sort of wildcard pattern that tells Spring to apply this Aspect to any method whose name starts with 'set', that has void as return type and that has exactly one argument. Pretty effective. This example illustrates very well what a pointcut is I would say, it is the general definition of _where an aspect should be inserted._ You might also say that a pointcut implicitly defines a collection of Join Points.

Working with AOP requires you to create a special configuration class like this:

```
@Configuration
@EnableAspectJAutoProxy
@ComponentScan(basePackages="com.example.bla")
public class AspectConfig {

	// It is common practice to leave the body empty
}
```

The @EnableAspectJAutoProxy configures Spring to use @Aspect. It enables AnnotationAwareAspectJAutoProxyCreator, a BeanPostProcessor. You must enable the whole aspect thing like this others no aspect things will be done. There are xml alternatives for it, and in Spring Boot you can omit it as Spring Boot automatically activates it. This @EnableAspectJAutoProxy is the only annotation that does what it does, there are no slightly different alternatives.

Subsequently, you must import the Aspect configuration class in the main configuration class (although I think you can circumvent this by supplying the aspect config class as argument when the ApplicationContext is created).

```
@Configuration
@Import(AspectConfig.class)
public class MainConfig{
	@Bean etc
}
```

Under the hood, any class containing @Aspect annotated methods (or methods that have been singled out by an AOP pointcut expression in the AspectConfig @Configuration @EnableAspectJAutoProxy class) will be converted to a proxy, either JDK or CGLib, by some BeanPostProcessor during the initialization phase. See 1.5.3. The so-called 'weaving' is basically done by a BeanPostProcessor and results in a proxy being returned by the method that implements BeanPostProcessor.

#### Retrieving information about Join Points

In the bean that defines the advice method (I don't know if this is a correct term) you can do a trick, namely the following:

```
@Aspect
@Component
public class PropertyChangeTracker {
	private Logger logger = Logger.getLogger(getClass());

	@Before("execution(void set*(*))")
	public void trackChange(JoinPoint point){
		String methodName = point.getSignature().getName();
		Object newValue = point.getArgs()[0];
		logger.info(methodName + " about to change to " + newValue + " on " + point.getTarget());
	}
}
```

Btw I wondered why @Aspect as annotation itself doesn't imply @Component automatically. What I understand from the tutorial is that @Aspect is not a Spring but an AspectJ annotation, which makes it understandable that it hasn't implemented @Component.

Specific information about the context where the @Aspect is applied is collected and can be used, in this case to make it part of a log.

### 1.6.3 Use AOP Pointcut Expressions

Spring AOP uses AspectJ's pointcut expression language. Spring AOP supports a practical subset of this language.

#### Common Pointcut Designator

- execution([method pattern])
The method must match the pattern

- chaining is possible using &&, ||, !
Like: execution([pattern1]) || execution([pattern2]) 

The pattern must be read from right to left. The elements that must be represented are arguments, methodname and returntype. Class/packagename is optional, as is access modifier.

There are two types of wildcards, namely `*` and `..`. The former stands for exactly one element, the later for zero or more elements. `*` can be used as a wildcard for a complete name or part of a name.

If you add the optional package/class name, with or without the use of `*`, you must work with FQN's and make the method name part of the FQN.

#### Examples

`execution(* rewards.restaurant.*Service.find*(..))`
Designate any method starting with find from a class in rewards.restaurant package whose name ends with Service. The method can have 0 or more arguments and can return any type, including void.

`execution(void send*(rewards.Dining))`
Any method that has one argument of type rewards.Dining, a name starting with 'send' and a void return type.

`execution(* send(*))`
Any method named send that has one argument. Return type can be anything.

`execution(* send(int, ..))`
Any method named send which has int as its first argument type.

`execution(void example.MessageService.send(*))`
Any method named send that sits in a class that _implements example.MessageService interface_, has one argument and void return type. Not that this is powerful, you can add an aspect to typical methods of (functional) interfaces that are implemented on different places in your application.

Note that return types and argument types need fully qualified names, while method name itself doesn't. The return type and argument type can reside in other packages, libraries, while the method by default can not.

#### Selection based on annotation

You can also select all methods that have a specific annotation:

`execution(@javax.annotation.security.RolesAllowed void send*(..))`
Any method with the RolesAllowed annotation from the specific package javax.annotation.security. The method name must start with send, have void return type and can have any number of arguments.

Again, use the FQN, this time of the annotation.

#### Advanced ones with package names

`execution(* rewards.*.restaurant.*.*(..))`
There is one directory between rewards and restaurant. The method must be in a class that sits in package `rewards.*.restaurant`, because you need one asterisk for the class and one for the method.

`execution(* rewards..restaurant.*.*(..))`
There are zero or more directories between rewards and restaurant. I find this one odd but it works (didn't try). 

`execution(* *..restaurant.*.*(..))`
There must be at least one directory before restaurant. The method must sit in a class that sits in a package whose FQN ends with restaurant.

### 1.6.4 Explain different types of Advice and when to use them

There are five advice types:

- @Before

If a @Before advice throws an exception, the target method will not be called. This is valid use of the @Before advice.

- @AfterReturning

Advice will only be executed if method executes successfully. Advice can use the return value of the method. This advice type does not only have a pointcut expression to match the method, but can also have an _optional_ second argument that matches the return type in a more sophisticated way. Example:

```
@AfterReturning(value="execution(* service..*.*(..)), returning="reward")
public void audit(JoinPoint jp, Reward reward){
	auditService.logEvent(jp.getSignature() + "returns the following reward object: " + reward.toString());
} 
```

Somehow the returntype must be Reward. The benefit of this construction is that you can select only those methods with a specific return type, and at the same time you can access that return value. If you are not interested in this return value, you can omit the 'returning' attribute.

- @AfterThrowing

Advice only executed if method throws exception. You will, similarly to the previous advice type, have access to the exception. Example:

```
@AfterThrowing(value="execution(* *..Repository.*(..))", throwing="e")
public void report(JoinPoint jp, DataAccessException e) {
	mailService.emailFailure("Exception in repository", jp, e);
}
```

The exception must be of type DataAccesException to activate the advise. If this exception is thrown an email is sent.

- @After

It doesn't matter with this one if the method was successful or not. Has just one pointcut expression, just like @Before.

- @Around

Most powerfull and most dangerous. It is versatile, can be used instead of @Before or instead of @After. Better use it only if you want both. Important: this is the only advice type that is responsible for delegating to the target method. You must include a .proceed() method in your advice, you cannot omit it but you can eventually use it multiple times. This method exists in the class ProceedingJoinPoint, this class is the type of the single argument you use. The proceed() method actually triggers the execution of the target method, so you can use it as a divider between before and after. Without proceed() the target method is not executed at all. This is all specific to @Around.

Note that the return type is Object and not void like the others. 

Example where you see the use of .proceed(). Note that the whole 'after' part is skipped if value already exists.

```
@Around("execution(@example.Cacheable * rewards.service..*.*())")
public Object cache(ProceedingJoinPoint point) throws Throwable {
	Object value = cacheStore.get(CacheUtils.toKey(point));

	if (value!=null) return value;

	value = point.proceed();  // proceed() runs the target method. 
	cacheStore.put(CacheUtils.toKey(point), value);
	return value;
}
```

#### Limitations

- Spring AOP can only advise non-private methods
- Can only apply aspects to Spring Beans
- Method calls within the same class will not be advised

About the latter: if method A in a class calls method B, that sits in the same class, advice will never be executed for B. The call must come from outside.





