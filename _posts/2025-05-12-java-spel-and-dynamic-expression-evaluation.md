## Java, SpEL and dynamic expression evaluation

I'm working on Spring, planning to do Spring Certified Professional / 2V0-72.22 from Broadcom/VMWare. I did a mock exam on Udemy and got 60%, think I need 77% or so to pass. It is doable, easier than Java SE 11. 

### SpeL

While learning, I stumbled upon the Spring expression language. Within Spring, you use it to add expressions and property values to your configuration. It can be added to the xml configuration file like this example from [Baeldung](https://www.baeldung.com/spring-expression-language#referencing%20a%20bean):

```
<bean id="spelOperators" class="com.baeldung.spring.spel.SpelOperators">
   <property name="equal" value="#{1 == 1}"/>
   <property name="notEqual" value="#{1 lt 1}"/>
   <property name="greaterThanOrEqual" value="#{someCar.engine.numberOfCylinders >= 6}"/>
   <property name="and" value="#{someCar.horsePower == 250 and someCar.engine.capacity lt 4000}"/>
   <property name="or" value="#{someCar.horsePower > 300 or someCar.engine.capacity > 3000}"/>
   <property name="addString" value="#{someCar.model + ' manufactured by ' + someCar.make}"/>
</bean>
```

Typical use in Spring is as argument for annotations:

```
@Value("#{2 > 1 ? 'a' : 'b'}") // evaluates to "a"
```

The thing to remember is that, when you use SpEL this way, you need to wrap expressions in ```#{...}```. And this one needs to be wrapped in double quotes. Another thing is that, when you want to refer to properties as defined in configuration files, you can insert them using ```${...}```. This notation is actually not a SpEL notation, just a more general placeholder. 

An expression can contain a property but not vice versa. If you want to include object properties, you separate object variable and property name by a dot like ```someCar.model```. Nothing with ${}. You can access nested object properties as well, like ```$Value("#{someCar.engine.capacity}")```. 

### Dynamic expression evaluation

Given that SpEL expressions are written as strings, I asked Perplexity if it was possible to create a dynamic sort of Java. You can generate a string during runtime via some sort of input and evaluate as a SpEL expression, which is a very un-Java way to do things (I have not introduced myself to reflection yet). In its answer, Perplexity wrote the following:

_"Allowing users to submit arbitrary SpEL expressions is dangerous and can lead to serious security vulnerabilities, such as code injection attacks. Modern Spring versions disable certain risky features by default, but you should never evaluate untrusted user input without strict validation and proper sandboxing."_

The things you can do with SpEL that can be unsafe are creating new instances of objects and calling methods on it. I might do some experiments with it. For now I have a bit more idea of what SpEL is.





