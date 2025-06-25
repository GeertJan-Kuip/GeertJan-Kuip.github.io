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

From the documentation: _The org.springframework.context.ApplicationContext interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on the components to instantiate, configure, and assemble by reading configuration metadata._

## Objective 1.2 Java Configuration

### 1.2.1 Define Spring Beans using Java code

If you define Java beans without an xml file, there are two options:

- Annotation-based configuration. This requires the use of annotations like @Autowired and @Component on the application's component classes
- Java-based configuration. A class annotated with @Configuration contains code and annotations that define the beans and the wiring. The @Beans methods act as static factory methods.

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

One @Configuration class can have an @Import annotation to be used for importing another @Configuration class. See [example](https://docs.spring.io/spring-framework/reference/core/beans/java/composing-configuration-classes.html). When constructing the ApplicationContext, it now is enough to have only the configuration class as argument that has imported the other configuration class.





### 1.2.4 Handle Dependencies between Beans

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

Objective 1.3 Properties and Profiles
1.3.1 Use External Properties to control Configuration
1.3.2 Demonstrate the purpose of Profiles

1.3.3 Use the Spring Expression Language (SpEL)

The Spring Expression Language is quite extensive. The basic usage is something like this:

```
@Value("#{ systemProperties['user.region'] }")
```



Objective 1.4 Annotation-Based Configuration and Component Scanning

1.4.1 Explain and use Annotation-based Configuration

Annotation-based configuration means that beans are being found by scanning

1.4.2 Discuss Best Practices for Configuration choices

1.4.3 Use @PostConstruct and @PreDestroy

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

1.4.4 Explain and use “Stereotype” Annotations

The most important stereotype annotations are Stereotype annotations are @Component, @Controller, @RestController, @Service, @Repository and @Configuration. They all indicate that the annotated class is a bean.

Objective 1.5 Spring Bean Lifecycle
1.5.1 Explain the Spring Bean Lifecycle
1.5.2 Use a BeanFactoryPostProcessor and a BeanPostProcessor
1.5.3 Explain how Spring proxies add behavior at runtime
1.5.4 Describe how Spring determines bean creation order
1.5.5 Avoid issues when Injecting beans by type
Objective 1.6 Aspect Oriented Programming
1.6.1 Explain the concepts behind AOP and the problems that it solves
1.6.2 Implement and deploy Advices using Spring AOP
1.6.3 Use AOP Pointcut Expressions
1.6.4 Explain different types of Advice and when to use them