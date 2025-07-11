# Spring Boot web application

[Module 4](https://spring.academy/courses/spring-boot/lessons/spring-boot-web-app-intro) of the Spring Boot part of the course is about creating a simple web application focussed on REST. It covers various topics, namely dependencies and writing the pom file, the structure of the MVC model, the specifics of the @(Rest)Controller class and the @GetMapping method, message convertors (like Jackson), using another webserver than Tomcat and Spring Boot Developer Tools. It ends with a small program that is packaged as a fat jar and can be run from the command line.

## Spring Web Introduction

### Web Servlet vs Web Reactive

The video starts with a short explanation on the differences between two types of Spring MVC applications, namely Web Servlet and Web Reactive. It is not clear to me what all the terms and names mean (Non-blocking approach, Netty, Undertow, reactive programming). The main point is that the tutorial is about the first, more traditional approach, Web Servlet. The term Web Flux is also used, this is a non-blocking Spring Framework.

### Traditional or Embedded Servlet Container

Spring Boot supports embedded servlet containers, which means you can create a fat JAR that can be run on its own, as it includes a web server. Spring Boot can also implement traditional structure needed by web containers. In this case you'll create a WAR package. The tutorial is about the embedded approach with JAR, but later on, there will be some info on WAR and what it means for the entry class and main method in your application.

### Dependencies

You need to add one starter to make a simple web application work. It has many transitive dependencies, like Jackson and Tomcat:

```
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

### Spring Boot creates MVC components

A diagram is being shown with all components of the MVC model as implemented by Spring Boot. The only components you need to create yourself are the controllers and views. 

The MVC model has a central object Dispatcher Servlet. It receives the URL request and sends back the response. The Dispatcher Servlet delegates to the Handler Mapping object, the Handler Adapter object, the Message Converter and the View Resolver. Things that should be done are finding out what the user request is (I suppose a combination of the path and the http request, it is done by the Handler Mapping), what the content of the response should be (here the controller comes in, but sits behind the Spring-managed Handler Adapter), and the formatting of the response (Jackson, templates, depending whether it is REST or not etc). The Spring Boot programmer must supply the 'view' here.

## Simple RESTful controller for GET requests

This video tutorial deals mainly with the @RestController class and the GET method within it.

### Using @ResponseBody or @RestController

A class annotated with @Controller is a component and will be found by component-scanner. You can use @Controller but alse @RestController, which combines @Controller with @ResponseBody. The @ResponseBody annotation is used when you want to return JSON or xml, not a rendered view. @ResponseBody is what makes a method in controller a REST method.

The example given is this:

```
@Controller
public class AccountController {

	@GetMapping("/accounts")
	public @ResponseBody List<Account> list() {...}
}
```

The @ResponseBody, together with @GetMapping, makes the list() method a RESTful method. If any method is a RESTful method, just use @RestController instead of @Controller.

Be aware that without @ResponseBody/@RestController, Spring will try to return a view with templates etc, not data.

The tutorial assumes that @RestController is being used from now on.

### The Handler Adapter

In the MVC structure there is the Handler Adaptor. Its function is to collect and structure all the information of the url request. This allows for the passing of this information to the method by injecting it in the form of arguments. There is a ton of information that can be injected and there is a set of usefull annotations for it. 

#### Injecting method arguments

In this environment, where an http request has been made, you can ask Spring to insert arguments with information about the user, the http request or the path. Simple example where Spring injects user info in the form of the Principal bean (some Spring defined bean):

```
@RestController
public class AccountController {

	// retrieve accounts of currently logged-in user
	@GetMapping("/accounts")
	public List<Account> list(Principal user){
		...
	}
}
```

Other beans you can use as argument are HttpServletRequest, HttpSession, Principal, Locale, etc. In the [Spring reference](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html) you find a long list. There are a lot.

#### Request parameters as method arguments

The following sample illustrates the use of the @RequestParam annotation. It lets you grab an argument from the http request, or the calling url. Let's say it is this:

```
http://localhost:8080/account?userid=1234
```

To fetch the userid parameter, you do the following with @RequestParam:

```
 	@GetMapping("/account")
	public List<Account> list(@RequestParam("userid") int userId){
		// now you have userId. It has been cast to int, it is String by default.
	}
```

#### Extracting path elements

Similar to @RequestParam is @PathVariable. Say you have a calling url with this path:

```
http://localhost:8080/accounts/98765
```

To fetch the account id stored in this path, you do the following with @PathVariable and a named placeholder:

```
 	@GetMapping("/accounts/{accountId}")
	public Account find(@PathVariable("accountId") long userId){
		// now you have accountId. It has been cast to int, it is String by default.
	}
```

#### Request parameter AND path element

The following shows how to get both. Say this is the calling url:

```
http://localhost:8080/accounts/1234?overdrawn=true
```

Then this fetches both:

```
 	@GetMapping("/accounts/{userId}")
	public List<Account> list(@PathVariable("accountId") long userId, @RequestParam boolean overdrawn){
		// now you have both in the right type.
	}
```

#### A complex example

Combine multiple argument types.

```
	@GetMapping("/orders/{id}/items/{itemId}")
	public OrderItem item(@PathVariable("id") long orderId,
				@PathVariable int itemId,
				Locale locale,
				@RequestHeader("user-agent") String agent) {..}
```

#### Optional request parameter

You can give @RequestParam an attribute 'required' and set it to false. If the request parameter is missing, the value will be set to null. Note that in the example below, the value is cast to Integer and not int. This is because Integer can be null and int can't.

```
http://.../suppliers?location=12345
```

```
	@GetMapping("/suppliers")
	public List<Supplier> getSuppliers(
				@RequestParam(required=false) Integer location, // Null if not specified
				Principal user,
				HttpSession session ) {..}
```

## Message Converters

Assuming we want to send the client JSON or other data, we have the problem that while we work with Java objects, the client wants a JSON or XML object, as indicated in an 'Accept' header. This requires conversion.

Here is where the Message Converter component in the Spring Web model, or the HttpMessageConverter object, comes in. The Message Converter is activated because of the @ResponseBody annotation and it converts Java objects into JSON or XML objects that are the body of the response we send. Jackson is a typical default library being used. GSON is an alternative but not the default. 

@ResponseBody does not only activate HttpMessageConverter, it also sort of deactivates ViewResolver and View.

There are actually multple message converters available so Spring can deal with any requested MIME type. It will simply look for the best converter for the job.

### "Accept" Request Header

The url request from client contains an 'Accept' header. If this is either 'application/xml' or 'application/json' we can send these types as the body of the response because of the HttpMessageConverters that Spring utilizes. We only need to tell what object to convert. If it is something else, Spring will look for another converter.

The good thing is that Spring takes care of the conversion based on the accept header. We don't have to care about it.

### Customizing response with ResponseEntity

ResponseEntity is a class with a Builder class inside that you can use to create a custom Response object for in the GET method. This might be handy if you want full control over the response. The difference with not using it is that you do not only provide the body ("payload") but also the headers. Example:

```
ResponseEntity<String> response = ResonseEntity.ok()
					.contentType(MediaType.TEXT_PLAIN)
					.body("Hello Spring");
```

It can be combined with code that specifically creates the body content.

## Packaging

By default Spring Boot starts up an embedded web container (Tomcat). This means you can run the web application from the command line. This is a very good thing, it minimizes dependencies.

### Using Jetty instead of Tomcat

If, for some reason, you do not want to use Tomcat but, for example, Jetty, you must change the pom file. Spring will notice that Tomcat is not on the classpath anymore while Jetty is and act accordingly. Jetty is detected and used. This is the pom:

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

### Running within a web container (traditional)

While this is seen as an outdated approach, you might need it somewhere. If you create a Spring Boot application that runs in an existing web container, you need to make adjustments to your Application entry class. It must be extended with SpringBootServletInitializer, which is a subclass of WebApplicationInitializer, which is called by the web container. 

```
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

	// specify the configuration classes to use
	protected SpringApplicationBuilder configure(
			SpringApplicationBuilder application) {
	
		return application.sources(application.class);
	}
}
```

The artifact type of your package must be WAR, which you must tell your pom file as well.

### The hybrid variant

It is possible to write your entry class in a way that the application can both be used as a WAR and a JAR. You take the code from the previous example and add a main file. Spring will understand what to pick, depending on it being a WAR or a JAR:

```
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

	protected SpringApplicationBuilder configure(
			SpringApplicationBuilder application) {
	
		return application.sources(application.class);
	}

	public static void main(String[] args){
		SpringApplication.run(Application.class, args);
	}
}
```

### WAR and Tomcat conflicts

Both JAR and WAR can be run stand-alone provided that they have embedded Tomcat (or Jetty). A problem arises when you make a WAR file with embedded Tomcat and try to run it inside a traditional web container. The included Tomcat might conflict with the webserver already in the container, if this is also Tomcat.

What you should do is to mark the Tomcat dependencies as _provided_ when building WARs for traditional containers. It means you will rely on the Tomcat version provided by the container and not bring your own Tomcat.

### Fat WAR

If you build JAR files for a Spring Boot project, two versions will be created, a fat one and a skinny one. The same is true when you build WAR files for a Spring Boot project.

### Spring Boot developer tools

When you change the code of your Spring application and want to run it, starting things up takes a lot of time. To make this faster, Spring has created a devtools package that somehow creates two separate classloaders, on for your own code and one for the rest (Spring, libraries). Now when you want to update your application somehow only your own classes will be reloaded, so not everything has to go through the loading process.

This feature is automaticaly disabled when Spring thinks that the app is running in "production". According to the video, this is when the app is run by the command line `java -jar`. Needs a bit more clarification.

To add devtools do this in pom:

```
<dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
</dependencies>
```

The `<optional>` tag prevents prevents devtools from being transitively applied to other modules that use your project.

## Small sample project

A small project is created. A pom is written, a class 'RewardController' is created, some entries to application.properties are being written, and the entry class 'Application' is being created.

### Maven pom.xml

The dependencies required are:

```
spring-boot-starter-parent   // for consistent versioning
spring-boot-starter-web  
spring-boot-maven-plugin    // fat jars
```

### RestController

This is the class code given in the tutorial:

```
@RestController
public class RewardController {
	private RewardLookupService lookupService;

	// constructor dependency injection
	public RewardController(RewardLookupService svc){  
		this.lookupService = svc;
	}

	@GetMapping("/rewards/{id}")
	public Reward show( @PathVariable("id") long id) {
		// returns reward object, converted by Message Converter automatically
		return lookupService.lookupReward(id);  
	}
}
```

### application.properties

Even without properties it would work. But to get familiar with application.properties, here some innocent ones:

```
#application.properties
server.port = 8088      // default is 8080
server.servlet.session.timeout=5m   // 5 minutes
```

### Application class

Extremely basic. But it works, there is a running program, although the injected class in @RestController is still missing. The tutorial assumes it is there, probably from earlier excercises.

```
@SpringBootApplication
public class Application {

	public static void main(String[] args){
		SpringApplication.run(Application.class, args);
	}
}
```





