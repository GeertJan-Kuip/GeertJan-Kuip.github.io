# Spring exam transaction management

I'm not doing things in the order of the video tutorial but transactions are newer for me so I do them first. I do not follow the order of the VMWare document describing the contents of the exam, these are just notes beased on the video tutorial.

## Setting up

There are 3 steps to take to implement transactions:

- Declare a PlatformTransactionManagerBean
- Declare the transactional methods with annotations and/or programmatic
- Add @EnableTransactionManagement to a configuration class

PlatformTransactionManager is the base interface, several implementations are available, like:

- DataSourceTransactionManager
- JmsTransactionManager
- JpaTransactionManager
- JtaTransactionManager
- WebLogicJtaTransactionManager
- WebSphereUowTransactionManager

### Declare a PlatformTransactionManagerBean

You do this by creating a new bean method in the configuration class that creates a class of reference type PlatformTransactionManagerBean:

```
@Configuration
@EnableTransactionManagement  // essential
public class AppConfig{

	@Bean
	public PlatformTransactionManager transactionManager(DataSource dataSource){
		return new DataSourceTransactionManager(dataSource);
	}
}
```

The DataSource bean that you pass as argument (something JDBC related) must of course also be created. You do not have to create your own class DataSourceTransactionManager, as this class already exists. It is one of the implementations of PlatformTransactionManager, one that is fit for JDBC stuff.

### Declare the transactional methods

The most obvious way to do this is by using the @Transactional annotation. Example:

public class RewardNetworkImpl implements RewardNetwork {

	@Transactional
	public RewardConfirmation rewardAccountFor(Dining d){
		// atomic unit of work
	}
}

It is also possible to add @Transactional to the class as a whole. now any method in the class will inherit the transactions, with the configuration (attributes) that you give the class level annotation. To customize you can also override this class level attributes at the method level.

### Add @EnableTransactionManagement

The annotation is processed because transaction management is enabled by @EnableTransactionManagement on the @Configuration class.

## Propagation

Propagation is about nested transactions. 

By default, a nested inner transaction looses its transaction-like wrapping and becomes part of the outer transaction. So there is only one transaction then.

You can change this behaviour and let the inner transaction exist. In this scenario, the two transactions are fully independent and do not influence each others success or failing. The process is that the outer transaction starts, then will be suspended to make room for the inner transaction, and if that one is done (successful or not) it will resume.

There are 7 levels of propagation of which only 2 are part of the test, namely:

- @Transactional(propagation=Propagation.REQUIRED)
- @Transactional(propagation=Propagation.REQUIRES_NEW)

The first one is the default (everything runs in the current transaction, if there is no transaction when an @transactionl method is called a new one is created). The second one makes multipl independent transactions at the same time possible.

### Propagation rules are enforced by a proxy

In this example, the second propagation rule does not get applied because the method is called from within its class and doesn't go through a proxy (or 'interceptor').

```
public class ClientServiceImpl implements ClientService {
	@Transactional(propagation=Propagation.REQUIRED)
	public void update1(){
		update2();
	}

	@Transactional(propagation=Propagation.REQUIRES_NEW)
	public void update2(){}	
}
```

I had some trouble understanding this but what I understand is that Java proxies only intercept external, not internal, calls to proxied beans. So when method A calls method B in a proxied bean, whatever the proxy is, it won't be involved and this is true for all inner calls. 

What I misunderstood is that I thought that once a proxy was created, it would take over all the work of the original class and the original class would be out of work. That is not true. The original class is still available, and will be used in the case of inner method calls.

I asked ChatGPT if the Spring makers could have chosen to make every method call, including the inner method calls, go through a proxy. Chat answered that there is a limitation in Java itself that makes it impossible to intercept inner method calls by a proxy. It would have required a different implementation to make it work like this. It would require weaving, class transformation etc. Btw AspectJ does this, but apparently Spring doesn't.

Java introduced the proxy mechanism in version 1.3, before Spring, for all sorts of other reasons. Spring simply used it as a convenient method and accepted its limitations. Generally, Spring uses all sorts of available things, it doesn't go on its own creating new high tech.

## Rollback

By default, when during a transaction a checked exception is thrown, the transaction will commit anyway. On the other hand, when a RuntimeException is thrown, the transactions will be rolled back. Many Spring things throw RuntimeExceptions which is actually a good thing.

But you can configure the rollback part with an attribute added to any @Transactional annotated method. Two attributes can be used:

- @Transactional(rollbackFor=MyCheckedException.class)
- @Transactional(noRollbackFor={MyOtherUncheckedException.class, MySecondUncheckedException.class})

## Testing with @Transactional

By default, if you add @Transactional to a @Test annotated test class, the transaction will be rolled back. Makes sense. Be aware that only the outer transaction is rolled back, the innter transaction(s) aren't!

### @Commit

Using @Commit on a class annotated with @Test and @Transactional makes the transaction commit instead of being rolled back.




