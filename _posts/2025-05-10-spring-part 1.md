## Spring part 1

I bought ['Spring Start Here' from Laurentiu Spilca](https://www.amazon.com/gp/product/B09HJLK2BN/), I found the [Spring reference](https://docs.spring.io/spring-framework/reference/) and I used ChatGPT to get up and running with Spring. Below is just a grocery list of things I learned up till now. There might be mistakes in it but I tried to be precise:

### Spring context and beans

- IoC, ApplicationContext is name of the interface of the container.
- Generally, the term ‘Spring context’ is being used.
- ApplicationContext is a subinterface of BeanFactory.
- ApplicationContext is implemented by several classes. Which one you choose might have to do with the way you configure the beans.
- GenericApplicationContext is a class implementing ApplicationContext. It autodetects beans.
- The .getBean(String name, Class<T> requiredType) method creates an instance of a bean.
- You let Spring know what the beans are and how they relate (how they are ‘wired’) in four ways: (1) Xml based (2) Java based with @Configuration class in which you use the @Bean annotation, (3) Java based with ‘stereotype annotations’ and/or programmatically by using the .registerBean() method on the ApplicationContext instance. The latter method has 4 parameters.
- Xml is not used for new projects, more a legacy thing. You use <bean> tag to define beans and <property> to do the wiring/dependencies.
- Using (2), a class marked with @Configuration in which you provide @Bean annotated constructors for the classes Spring needs to control, gives more control over bean construction.
- These constructor methods in @Configuration are actually normal methods that have a name (can be any) and return the type that you want to construct.
- Stereotype annotations are @Component, @Controller, @RestController, @Service, @Repository and @Configuration. I might miss some. @Controller is the most basic. @RestController is a special case of @Controller, both denote classes where http requests are mapped.
- If you have a configuration class (@Configuration) you can add @ComponentScan as well to this class. This tells Spring to search for stereotype annotations to find beans that are not constructed in the configuration class itself.
- @ComponentScan can be given a parameter telling it in which package(s) to search for beans.
- The annotations @Configuration and @ComponentScan can be combined into one: @SpringBootApplication.
- @SpringBootApplication includes as well @EnableJpaRepositories.
- For regular beans there are two scopes, singleton and (forgot name). You can create multiple instances of a singleton scope if you provide more than one @Bean constructor method in the configuration class for that same bean.
- A bean created by a constructor in the configuration class is named after the method name of the constructor method that creates it. So two instances of the same Singleton bean cannot have the same name.
- Singleton beans are stored in a special object, DefaultSingletonBeanRegistry. This class has a field of type Map for it. If you create a singleton scoped bean that already exists, Spring will return the existing one.


### Proxy class

- Important term: proxy class. When Java creates a bean instance it constructs not the original Java defined object but a sort of enhanced object. This alteration/enhanced version of the class is the result of annotations like @Transactional, @Async, @Cacheable, @Secured and @Aspect. 
- This proxy object has its own .class definition that is stored in-memory.
- If the class to be proxied implements an interface, Spring uses JDK Dynamic Proxies. A proxy is generated based on the implemented interface.
- If no interface is implemented, Java uses CGLIB. The proxy is constructed as a subclass of its original class. This implicates that classes and methods of classes to be proxied this way cannot be final.
- JDK Dynamic Proxies uses Java’s built-in java.lang.reflect.Proxy class. If you check the name of such a proxy object it has the form of com.sun.proxy.Proxy123.
- CGLIB proxies use the CGLIB library. This is an older library, developed by a third party. The typical name of such a proxy object is com.example.MyService$$EnhancerBySpringCGILIB$$abc123.
- CGLIB stands for Code Generating Library.
- Since Spring 3, Spring has a preference for JDK proxies. Only if JDK proxy is impossible it reverts to CGLIB.
- CGLIB is reflection-heavy and therefore has slower startup and a bit more memory usage.
- ByteBuddy is a more up-to-date variant of CGLIB and will replace it.


### Webserver

- To start the TomCat webserver, you need to call SpringApplication.run(). Instead of TomCat you can set other webservers, Jetty and Undertow.
- TomCat is the default if you do not choose. If you want another one, you need to add the dependency for it in the pom.xml file and you need to exclude TomCat in the pom with the <exclusion> tags
- TomCat is written in pure Java and runs in the JVM. In Spring, TomCat is added as a library and just part of Spring running. So Spring and webserver are one application.
- A DispatcherServlet is created by the webserver when an http request comes in. It (1) searches for the controller bean (@Controller), it contacts the controller to do the mapping, and it resolves (is that the right word?) the view by finding and activating the template.
- @RestController combines @Controller and @ResponseBody.
- The template can be found in either resources/static or resources/template, depending on whether pages are built dynamically or not.
- If you want dynamic pages, you can include Thymeleaf in the pom.xml. Thymeleaf is a simple template engine. It can do conditionals, for-each etc. The th: tag is typical for Thymeleaf templates.
- @RequestMapping is the general annotation for methods in the controller class that return webviews. There is also @GetMapping, @PostMapping, @DeleteMapping and @UpdateMapping. 
- Curly braces are used inside the mapping path to capture dynamic values from the url. You can do a lot with regex, pattern matching, wildcards etcetera.


### Aspect Oriented Programming

- AOP stands for Aspect Oriented Programming. It lets you define code snippets and apply them to multiple methods. If, for example, you want a log that logs the convocation of certain methods, you can write the log method as an aspect and add it to the methods you want to log.
- To create an ‘aspect’, create a class and annotate it with @Aspect and @Component. It is important to note that @Aspect is not a stereotype annotation so you need to either give it @Component or construct it with @Bean in the configuration class.
- The method you want to be executed as aspect needs to be annotated by @Before, @After or (there is a third one). The parameter for this annotation tells to which methods the aspect method is to be added.
- The aspect can only be applied to methods within beans.
- The parameter is written in some code language, looks complicated but you can look it up if needed.
- Spring Aspects requires a dependency added to pom.xml, namely <artifactId>spring-aspects</artifactId>

- Java beans are often connected, ie a bean can contain another bean. 
- You can connect or ‘wire’ them in multiple ways, similar to the multiple ways in which you can define the Spring context.
- (1) Dependency injection. Via constructor or setter. The dependency is injected as parameter of the constructor or the setter method.
- If you use the dependency as parameter of the @Bean constructor method, you can simply use a setter method with the parameter. You do not have to create the dependency instance, Spring will either do it for you or grab the right instance from the DefaultSingletonBeanRegistry object.
- Important: Spring protects its singletons by return the existing instance if you try to create a new one while there is one already. If you have a constructor of bean A that calls the constructor of bean B as parameter, that constructor of bean B might  return the existing instance. This is actually very handy.
- (2) Add @Autowired to a field, assuming this field is of a bean-type. Upon construction this field will automatically be set to the instance of the specified bean type. Note: this is not very good practice, because you cannot make the field final. It has maintainability problems as well.
- Better practice is to use @Autowired on the constructor (the ‘real’ constructor in the component class). You add the annotation on a constructor that has the dependency as parameter/argument and voila. The field containing the dependency can now be made final (remember that an instance field can only be made final if it is initialized immediately, in an instance initializer block or in the constructor). 
- (3) Old school via xml file: within <bean></bean>, add <property name=”..” ref=”..”/>. This is called ‘wiring’, as opposed to ‘auto-wiring’.

### Database, repository

- Database use is simplified by Spring. You don’t have to create Connection, PreparedStatement, ResultSet etc.
- If you include spring-boot-starter-data-jpa to dependencies then you can create/define your own versions of interface JpaRepository (interface MyRepository extends JpaRepository). 
- If @ComponentScan is active, Spring will find these interfaces so you can use them. Structure is important: The configuration class must be in top-level package, the JpaRepository implementation(s) must be in subpackages. 
- You can add a parameter to @EnableJpaRepositories to let it know where to scan for JpaRepositories.
- They do not have to be annotated, it is enough if they extend JpaRepository.
- The @EnableJpaRepositories is automatically applied once Spring detects spring-boot-starter-data-jpa in the pom file.
- Based on such an interface, Spring under the hood creates a special type of bean, namely a JpaRepositoryFactoryBean. This is a proxy class that implements your interface. It does all the database operations, super powerful.
- A JpaRepository interface is used to define methods that do CRUD operations. There is a very compact syntax you can use.
- You can add a dependency in pom.xml for a database (h2, mysql, postgresql or whatever). Spring can initialize things. You configure the database in application.properties file.
- In-memory databases are even easier (h2 and derby).
- Spring uses Hibernate.
- Repository, Data Access Object and DAO are different names for the same thing.

