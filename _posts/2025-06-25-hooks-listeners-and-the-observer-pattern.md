# Hooks, listeners and the observer pattern

When studying Maven I was somehow in the dark about what it meant that Maven had 'hooks' that allowed plugins to add code to the Maven lifecycle. 'Hook' is a metaphor, I had no clue about the underlying implementation.

Equally in the dark I was about the concept of ActionListeners in Java Swing, at the start of my Java journey. Listeners had to be added and you needed to write some actionPerformed method. I made it work but in a sort of mechanical way, with no feel for the underlying implementation.

Today I ran into so called 'BeanFactoryPostProcessors' in Spring and I asked ChatGPT about it. I related the answer to earlier Maven discussion and now I have understood something that didn't come intuitively to me.

## Spring and BeanFactoryPostProcessor

```BeanFactoryPostProcessor``` is an interface. You implement it in a Bean and at a certain point in the Spring lifecycle, after all Bean information is gathered, Spring calls the interface method. This is the interface code:

```
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

You can use the postProcessBeanFactory() method to:

- Change property values defined in @Bean or XML
- Register additional bean definitions programmatically
- Modify scopes, init methods, dependencies
- Inject configuration from external sources

Internally Spring does this:

```
Map<String, BeanFactoryPostProcessor> postProcessors =
    beanFactory.getBeansOfType(BeanFactoryPostProcessor.class);

for (BeanFactoryPostProcessor pp : postProcessors.values()) {
    pp.postProcessBeanFactory(beanFactory);
}
```

The essential thing is that it is the application that calls all the .postProcessBeanFactory() methods from a central place. Personally I would probably never come up with the idea of some central broadcaster making a call to all who are able to listen to do their specific trick.

One interesting thing is that Spring finds the classes/beans with the specific interface by searching them based on `Class<T>` type: ```beanFactory.getBeansOfType(BeanFactoryPostProcessor.class)```.

## Maven

In Maven something very similar happens with plugins. In your plugin you implement a specific interface, knowing that on a specific point in the lifecycle the method belonging to that interface will be called. Here ChatGPT also used the term hook, which was the thing that triggered me.

## Swing

Swing, and I remember JavaScript as well, have the concept of ActionListeners. Implementing them means using an addListener method, if I remember well, and either implementing some interface and writing an ActionPerformed method or extending some Action class with methods similar to ActionPerformed. 

I asked ChatGPT if this thing with listeners could be called the observer pattern (yes) and if the hooks in Spring and Maven could be interpreted as an observer pattern as well. ChatGPT said no on the latter, arguing that there are some fundamental differences here between Spring/Maven and Swing, namely the following:

|Aspect|Observer pattern|Spring post-processor mechanism|
|----|----|----|
|Purpose|React to events dynamically|Customize lifecycle behavior at startup|
|Notification trigger|An event occurs (e.g., button click)|The framework itself starts a phase (e.g., bean definition processing)|
|Runtime behavior|Ongoing, reactive|One-time, controlled startup|
|Unsubscribing/changing listeners|Dynamic|Static (all post-processors run)|

This is a good assessment but for me the similarity is still striking. It is the fact that there is some central method that does a sort of broadcast to many methods at once, all these methods have the same name but different content. 

## Concluding

To get a more intuitive feel for both 'hooks' and 'listeners', I'm gonna put them in the same mental space. I drop the term 'Observer pattern' and use 'Broadcast pattern' for it. The idea of code that 'listens' or 'observes' or has 'hooks' doesn't ring a bell for me because I always tend to think of code as something that has an active role, that is doing something, instead of waiting for or lilstening to something. The method that does the call is the proponent in my view, and I want him to be the central part of my understanding.