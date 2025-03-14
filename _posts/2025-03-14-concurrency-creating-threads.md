## Concurrency - creating threads

This topic, [chapter 18](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1), is the hardest I encountered. I'm planning to write four blog posts on it, this is the first one about creating threads. During Java's development the original way of managing threads using the Thread class was replaced by the use of the Concurrency API. Although the latter is the predominant way to do things now, both need to be learned.

### Using the Thread class

Creating and running a new thread requires two methods: void _start_(), which is a member of the Thread class, and void _run_(), which is found in functional interface Runnable. Thread implements Runnable, and the run method needs to be overridden/customized to get result you want. This does not mean that you can start a new thread by calling _run_(). Doing so results in the code being executed within the existing thread, not in a new one. It is _start_() that creates a new thread and runs the code of the run method.

You can create a new thread in three ways:

#### 1 - Extend the Thread class, overriding _run_():

```
public class CustomThread extends Thread {

  @Override
  public void run() {

    -- code to be executed in the new thread
  }
}

-- to start the new Thread from somewhere:
new CustomThread().start();
```

#### 2 - Create a class with Runnable implemented

```
public class MyClass implements Runnable {

  @Override
  public void run() {

    -- code to be executed in the new thread
  }
}

-- to start the new thread:
new Thread(new MyClass()).start();
```

#### 3 - Create a lambda

```
Runnable runnable = ()->System.out.println("This is my world");
new Thread(runnable).start();

--or

new Thread(()-System.out.println("This is my world")).start();
```

#### Thread.sleep()

Thread.sleep() can be used to pause a thread. It throws an InterruptedException when interrupted, which mean that you have to deal with that using a try-catch or by changing the signature of the enclosing method.


### Using the Concurrency API

The Concurrency API has three similar interfaces, namely Executor, ExecutorService and ScheduledExecutorService, that can be used to manage threads on a higher abstraction level. In the inheritance hierarchy Executor sits on top and has just one method, _execute_(), ExecutorService is its subinterface and has more methods, and ScheduledExecutorService is a subinterface of ExecutorService and has four extra methods related to scheduling.

Behind these three interfaces, of which Executor is not useful, are eight object types that can be created by static factory methods from the Executors class. These eight types create either single thread pools or single threads in different variants. The test requires that you know five of them.

Creating Executors/Thread pools in its basic form goes as follows:

```
public class MyClass {

  public static void main(String[] args){

    ExecutorService service = null;

    try{
      service = Executors.newSingleThreadExecutor();
      service.execute(()->System.out.println("Hi"));
      service.execute(()->System.out.println("Hello"));
    } finally {
      if (service!=null) service.shutdown();
    }
  }
}
```

The basicness of this example is in the fact that a newSingleThreadExecutor object is created, which is a simple one that creates just one thread. The lambdas will thus be executed sequentially. Furthermore, the execute() method is being used with Runnables. The execute() method does not return anything and neither do runnables.

More interesting is the use of _submit_() instead of _execute_(). Submit is a method of ExecutorService, not of Executor, and it differs in two respects. Firstly, it allows a Callable as argument. The code of Callable is:

```
@FunctionalInterface
public interface Callable<V> {
  V call() throws Exception;
}
```

It returns a generic value and it throws an exception if unable to return a result. Secondly, _submit_() returns a result of interface type Future\<?\>. This object can be used to retrieve the return value of Callable and to get information about the status and progress of the task that was submitted. The following code creates something with multiple threads, submit(), a Callable object and a Future object.

```
public class MyClass {

  public static void main(String[] args){

    ExecutorService service = null;
    
    Callable<String> callable01 = ()-> "Result 1";
    Callable<String> callable02 = ()-> "Result 2";
    Callable<String> callable03 = ()-> "Result 3";

    try{
      service = Executors.newCachedThreadPool();
      Future<String> futur01 = service.submit(callable01);
      Future<String> futur02 = service.submit(callable02);
      Future<String> futur03 = service.submit(callable03);
      System.out.println(futur01.get());
      System.out.println(futur02.isDone());
      System.out.println(futur03.isCancelled());	
    } finally {
      if (service!=null) service.shutdown();
    }
  }
}

-- Result 1
-- true
-- false
```

Note that the first line in the main method declares the ExecutorService variable to be null. This is what the book does, and I noticed that if I only declare the variable without assigning a value, the compiler protests because of the finally block ("Variable 'service' might not have been initialized"). Omitting the initialization at the begin is not an option either, because then the declaration/initialization takes place in the try block, which makes it a local variable not accessible by the finally block.

### invokeAll() and invokeAny()

These two methods belong to the ExecutorService interface and can also be called for ScheduledExecutorService type objects. They allow you to submit a list of Callables to the service, which will divide them over the threads. With invokeAll(), instead of one Future object as return value you get a list of Future objects, in the same order as the list of Callables. Only after all Callables are either finished or have thrown an exception the program will proceed. With invokeAny(), it is about the task that executes fastest. Only a single Future object is returned, of the Callable that is finished first. All other tasks are cancelled.

Below is sample code using invokeAll():

```
public class MyClass {

  public static void main(String[] args){

    ExecutorService service = null;
    
    Callable<String> callable01 = ()-> "Result 1";
    Callable<String> callable02 = ()-> "Result 2";
    Callable<String> callable03 = ()-> "Result 3";

    try{
      service = Executors.newCachedThreadPool();

      List<Future<String>> futurList = service.invokeAll(List.of(callable01, callable02, callable03));

      for (Future<String> f: futurList)
          System.out.println(f.get());

    } finally {
      if (service!=null) service.shutdown();
    }
  }
}

-- Result 1
-- Result 2
-- Result 3
```

Note that while the results of the tasks are printed in their order of submission, other output of the tasks comes in random order.

### Scheduled tasks

ScheduledExecutorService implements ExecutorService and is meant to create scheduled tasks, either on a single thread or on multiple threads. The object types related to it are SingleThreadScheduledExecutor and ScheduledThreadPool. Both can be created by the Executors class. 

ScheduledExecutorService has four extra methods, one of them an overloaded one. All of them return a ScheduledFuture type object, which is a subclass of Future. As ScheduledFuture has no extra methods, it works the same as Future. The four methods of 
ScheduledExecutorService are:

- ScheduledFuture<?> **_schedule_**(Runnable command, long delay, TimeUnit unit)
- <V> ScheduledFuture\<V\> **_schedule_**(Callable\<V\> callable, long delay, TimeUnit unit)
- ScheduledFuture<?> **_scheduleAtFixedRate_**(Runnable command, long initialDelay, long period, TimeUnit unit)
- ScheduledFuture<?> **_scheduleWithFixedDelay_**(Runnable command, long initialDelay, long delay, TimeUnit unit)

The difference between the latter two methods is that scheduleAtFixedRate() has a fixed time interval between the start points of tasks, while scheduleWithFixedDelay() has a fixed interval between the completion of one task and the start of the next.

### List of executor objects 

The Executors class has static methods that can generate 8 different executor objects, five of them are on the test:

| Method | Description |
|---------|-------------|
|ExecutorService **_newSingleThreadExecutor_**() |single thread, unbounded queue, sequential processing in order of submission |
|ScheduledExecutorService **_newSingleThreadScheduledExecutor_**()|single thread, schedules commands after given delay or to execute periodically |
|ExecutorService **_newCachedThreadPool_**()|creates thread pool, new threads are created as needed and idle old ones are reused |
|ExecutorService **_newFixedThreadPool_**(int)|creates fixed number of threads operating of a shared unbounded queue |
|ScheduledExecutorService **_newScheduledThreadPool_**(int)|creates thread pool that can schedule commands to run after given delay or to execute periodically |

Calling newFixedThreadPool(1) is identical with calling newSingleThreadExecutor. The newCachedThreadPool creates a thread pool of unbounded size, so no task will have to wait. There is a danger of ending up with too many threads, and for long-lived processes, usage of this executor is strongly discouraged. Its use is for executing many short-lived tasks. 

### TimeUnit

Various methods use the enum TimeUnit for wait, delay, periodical execution etc. These are the TimeUnit values:

|Enum name|Description|
|----------|-----------|
|TimeUnit.NANOSECONDS |one-billionth|
|TimeUnit.MICROSECONDS|one-millionth|
|TimeUnit.MILLISECONDS|one-thousandth|
|TimeUnit.SECONDS||
|TimeUnit.MINUTES||
|TimeUnit.HOURS||
|TimeUnit.DAYS||

### Future methods

|Method name|Description|
|----------|----------|
|boolean **_isDone_**()|Returns true if task was completed, threw an exception, or was cancelled|
|boolean **_isCancelled_**()|Returns true if task was cancelled before completing normally|
|boolean **_cancel_**(boolean mayInterruptIfRunning)|Attempts to cancel task execution, true if successfully cancelled, false if it could not be cancelled or completed anyway|
|V **_get_**()|Retrieves result of task (return value Callable), waiting endlessly if not available|
|V **_get_**(long timeout, TimeUnit unit)|Retrieves result of task, waiting for the specified amount of time. If result is not ready when timeout is reached, a checked TimeoutException will be thrown|

### ExecutorService methods

|Method name|Description|
|----------|----------|
|void **_execute_**(Runnable command)|Executes a Runnable task at some point in future|
|Future<?> **_submit_**(Runnable task)|Executes a Runnable task at some point in future and returns a Future representing the task|
|\<T\> Future\<T\> **_submit_**(Callable<T> task)|Executes a Callable task and returns a Future|
|\<T\> List\<Future\<T\>\> **_invokeAll_**(Collection<? extends Callable<T>> tasks) throws InterruptedException|Executes given tasks and waits for all of them to complete. Returns list of Future instances, in the order of the original collection|
|\<T\> T **_invokeAny_**(Collection\<? extends Callable\<T\>\> tasks) throws InterruptedException, ExecutionException|Executes the given tasks and waits for at least one to complete. Returns a Future for the complete task and cancels the rest of the tasks|



