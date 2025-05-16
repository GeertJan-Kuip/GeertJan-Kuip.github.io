## Spring root annotations

The Spring framework kind of overwhelms me, there being so many ways to do the same thing without much overall structure guiding my thinking. One thing where this happens is with the annotations, of which there are a great many. 

My thought was that it might be better not to learn every annotation as a separate thing but to learn the composition/inheritance trees. Actually very similar to Java classes, where for example a ton of collection classes can be understood by knowing a set of basic interfaces like Map, List, Deque etc. 

### Root annotations

I asked ChatGPT to provide a list of all Spring annotations that do not have a Spring meta annotation, ie they do not inherit behavior from another Spring annotation. With this list, is the idea, you can compose all other annotations (whereby it is possible that these other annotations show behavior that cannot be explained purely by their meta annotations, although I would hope the makers of Spring are somehow rigorous in this).

Anyhow, the list:

#### Core Stereotype Annotations

- *@Component* - Marks a Spring-managed component (bean)
- *@Repository* - Specialized component for persistence layer
- *@Service* - Specialized component for service layer
- *@Controller* - Marks a web MVC controller

#### Core Configuration Annotations

- *@Configuration* - Indicates a configuration class
- *@Bean* - Marks a method as a bean producer
- *@Import* - Imports additional configuration classes

#### Dependency Injection Annotations

- *@Autowired* - Automatic injection of beans
- *@Qualifier* - Distinguish between multiple bean candidates
- *@Value* - Injects values from properties
- *@Primary* - Indicates a preferred bean when multiple candidates exist

#### Aspect-Oriented Programming (AOP)- *@Aspect* - Defines an aspect class

- *@Before, @After, @Around, @AfterReturning, @AfterThrowing* - Advice annotations

#### Transaction Management 

*@Transactional* - Manages transaction boundaries (Note: may have some internal meta-annotations, but generally treated as base)

#### Scheduling

- *@Scheduled* - Marks scheduled methods

#### Others

- *@Scope* - Defines bean scope
- *@Lazy* - Lazy initialization
- *@PostConstruct and @PreDestroy* -  (Java standard, not Spring)