# Spring exam - Spring Core

Past month I did some other things (PowerShell scripting, Maven, JUnit, debugging) but now I want to proceed with Spring, getting the Spring Certified Professional certification this summer. I have the [exam guide] (https://docs.broadcom.com/doc/vmw-spring-professional-develop-exam-guide) and will use it as a roadmap to get famililar enough with Spring.

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

Inversion of Control and Dependency Injection are complementary. Dependency Injection means that the dependencies of a class (those objects that it works with and that have the form of member fields) are either initialzed by constructor arguments, by arguments belonging to a factory method or by properties that are set on the object instance right after construction of it. 

In all of these cases, the dependency is created outside of the object instance itself. The opposite of DI would be if somewhere in a method or a constructor or a initialization block a 'new DependencyType()' or a 'DependencyType.get()' method was being used.

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


## Objective 1.2 Java Configuration





### 1.2.1 Define Spring Beans using Java code


1.2.2 Access Beans in the Application Context
1.2.3 Handle multiple Configuration files
1.2.4 Handle Dependencies between Beans
1.2.5 Explain and define Bean Scopes
Objective 1.3 Properties and Profiles
1.3.1 Use External Properties to control Configuration
1.3.2 Demonstrate the purpose of Profiles
1.3.3 Use the Spring Expression Language (SpEL)
Objective 1.4 Annotation-Based Configuration and Component Scanning
1.4.1 Explain and use Annotation-based Configuration
1.4.2 Discuss Best Practices for Configuration choices
1.4.3 Use @PostConstruct and @PreDestroy
1.4.4 Explain and use “Stereotype” Annotations
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