## ExecutorService

Today I did an online Udemy test for 1Z0-819 and got 66% score. Half of the wrong answers were in the categories 'concurrency' and 'security', which actually delighted me, as I haven't put much time in them relative to other topics. It means I covered the other topics reasonably well.

To get better with concurrency, I need to try to understand a bit better how multithreading works. Usually it helps to try to get a bit more in-depth than required, that helps to get a better feel for it. Here some thoughts.

### Why are ExecutorServices created in a try-finally block?

In a previous post I put this code, which is a variation of code samples in [the book](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1):

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

A try-finally construct is used, and I wondered why there was no catch block and why one would not choose a try-with-resources construct. I learned a few things:

- In Java 11, ExecutorService did not have AutoCloseable implemented which makes it impossible to put the initialization of ExecutorService in the resource specification section of try-with-resources (I tried, it results in compiler error). 
- Since Java 19 AutoCloseable is implemented. Its default close() method calls the shutdown() method of ExecutorService.
- Perplexity argued that try-with-resources is inappropriate for this scenario, as it would close ExecutorService when the end of the try block is reached. By that time the tasks are probably still running (that's the whole point of asynchronous operations)
- Perplexity advised that if you want to use try-with-resources, you should use 'awaitTermination()' to prevent premature shutdown. Code example:

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    executor.submit(() -> System.out.println("Blocking task"));
    executor.awaitTermination(1, TimeUnit.MINUTES); // Required to avoid premature shutdown
}
```

Given these considerations, the try-finally construct is a good option and in Java 11 it is sort of the only option. Finally must be used to explicitly close the ExecutorService object (really important). Using .shutdown() works elegantly and is recommended. .shutdownNow() is for occassions where tasks become unresponsive, after the program has waited for some period. Perplexity gave this snippet:

```
executor.shutdown();
if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
    executor.shutdownNow(); // Force termination after timeout
}
```

This looks reasonable.

### Why no catch block?

What is mostly missing in sample code is the catch block. I asked Perplexity and it answered that exceptions in asynchronous tasks can better be handled within these tasks themselves. A task can involve a substantial piece of code in which exception handling is integrated. The try-finally construct is purely meant for proper shutdown of ExecutorService.

Nevertheless, there can be a reason to include a catch block, when it is related to the submit() method itself, when tasks are rejected. This is what Perplexity provides as sample:

```
try {
    executor.submit(() -> riskyOperation());
} catch (RejectedExecutionException ex) {
    // Handle case where executor is already shut down
} finally {
    executor.shutdown();
}
```








