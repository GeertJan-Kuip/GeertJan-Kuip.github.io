## Concurrency - safe code

"Writing thread-safe code is, at its core, about managing access to _state_, and in particular to _shared, mutable state_." This line is from Brian Goetz in his book "Java Concurrency in Practice." This post covers the Java methods dealing with shared, mutable state in programs with more than one thread.

Thread-unsafety means two things can go wrong. First, interleaving can take place whereby two threads call the same method. Even small methods (the typical example is var++) take a few steps leaving the door open for problems. Second, a thread can adjust state in a way that other threads will not be informed about that new state. The terms "happens-before relationship" and "memory consistency" are relevant here.

### Synchronized blocks

A classic way to create thread-safety is to create synchronized blocks. Code lines that change shared state are put within curly brackets and it is preceeded with keyword synchronized and some object between parentheses. This object is called a monitor and if you wondered why Object has the methods _notify_, _notifyAll_ and _wait_, it is because of this:

```
synchronized(someObject) {
  --code that can only be accessed by one thread at a time
}
```

The object can be any object, which makes it a bit strange. The main thing is that it must be the same object for every thread that wants to access the code. I asked perplexity and it told me that you should avoid strings and boxed primitives. Strings are in a string pool and some other code might have access to the same string for completely different reasons. Often, in an instance context, 'this' is used, which makes sense given the condition that every thread must deal with the same monitor object.

Synchronized guarantees not only that two threads are running the same code simultaneously, but also that every change of state is visible to all threads.

#### Synchronized instance method modifier

Similar to synchronized block. You add 'synchronized' as a modifier to an instance method (before return value) and the method becomes thread-safe. You do not have to provide a monitor, as it automatically synchronizes on the object itself.

#### Synchronized block in a static context

Which object should you use as monitor in a static context, given that it should be the same object for every thread requesting access to the synchronized block? The right answer is the class object. If you add synchronized as modifier to a static method you do not have to mention the monitor object, but if you write a synchronized block within a static method, you must do the following:

```
public class MyClass {

  public static void doSomething() {
    synchronized(MyClass.class) {
      -- thread safe code
    }
  }

}
```

It might look awkward but the MyClass.class object meets all the requirements. It is an object and it is the same everytime some thread wants to access the synchronized block.

### Atomic classes

These classes reside in the java.util.concurrent.atomic package. The three classes you need to know are AtomicBoolean, AtomicInteger and AtomicLong. You do not use regular operators on them (+, ++, = etc) but custom methods like below:

|Method name| Description|
|-----|-----|
|**_get_**()|Retrieves current value|
|**_set_**()|Equivalent to = assigment operator|
|**_getAndSet_**()|Sets new and returns old value|
|**_incrementAndGet_**()|Equivalent to ++val|
|**_getAndIncrement_**()|Equivalent to val++|
|**_decrementAndGet_**()|Equivalent to --val|
|**_getAndDecrement_**()|Equivalent to val--|

There are actually many more methods, you can do most things you would want to with Atomic classes.

### The Lock framework

This framework was introduced in version 1.5, as part of the Concurrency API, and has useful functionality that synchronized lacks. Instead of synchronizing on any object, you must now lock on an object that implements the Lock interface. Typical code with Lock looks like this:

```
Lock lock = new ReentrantLock();
try{
  lock.lock();
  -- thread safe code
} finally {
  lock.unlock();
}
```

The try-finally construct kind of guarantees that locks are released so that other threads can aquire it. The ReentrantLock class is one of the classes that implements Lock and I suppose it is one that works well in many situations.

Important: if you unlock a lock you don't have (you didn't call the lock method or it denied access when you tried), an IllegalMonitorStateException is thrown.

The following Lock methods are to be learned:

|Method|Description|
|----|----|
|void **_lock_**()|Requests a lock and blocks thread until lock is aquired| 
|void **_unlock_**()|Releases a lock|
|boolean **_tryLock_**()|Requests a lock and returns immediately. Returns a boolean to indicate if it acquired the lock| 
|boolean **_tryLock_**(long, TimeUnit)|Requests lock and blocks up to the specified time until lock is required. If the lock is not available within set period, returns false, otherwise true|

A lock can only be unlocked if it has been locked first. When using tryLock you should check whether the lock is acquired, because if so, you should call unlock, and if not, you should **_not_** call unlock. Therefore the book recommends if-try-finally-else:

```
Lock lock = new ReentrantLock();
if(lock.tryLock()){
  try{
    -- thread safe code
  } finally {
    lock.unlock();
  }
} else {
  System.out.println("I'm sorry, you didn't get in.");
}
```

This code prevents using unlock() without lock() and therefore IllegalMonitorStateException. You cannot call unlock just to be sure. Be aware that lock.tryLock() acquires the lock if it is available. You do not have to call lock.lock() again.

#### More on boolean _tryLock_(long,TimeUnit)

This is a hybrid of lock() and tryLock(). If the lock is available, it will acquire it immediately and return true. If the lock is unavailable, it will wait up to the specified time and if the lock is still not acquired then, it will proceed. With this overloaded tryLock variant you should use the if-try-finally-else as explained before, to prevent unlocking something that was never locked.

#### Obtaining and releasing a lock multiple times

You can obtain the same lock more than once but then you need to release it the same amount of times. Lock() (or tryLock()) and unlock() must be balanced.

#### Why Lock is better than synchronized

- Ability to request a lock without blocking (tryLock())
- Limiting the blocking period (tryLock(long, TimeUnit)
- A lock can be created with a fairness property, in which the lock is granted to threads in the order it was requested

### CyclicBarrier

If you have a list of tasks that you want to be performed by multiple threads, and you want those threads to wait for all the other threads before starting a next task, you can use the CyclicBarrier class. All threads stop at logical barriers. These are the ingredients:

- the number of threads involved (n)
- the list of tasks
- the points where the threads that finish a task early wait for the threads that finish later

The book has an example with 4 managers cleaning a pen where lions live. First they must remove the lions, then clean the pen, then move the lions back. You do not want a manager to bring lions back in the pen while another manager is still cleaning there. Noone should start a new task before everyone else is ready with the previous task.

Generally, you need a FixedThreadPool object ("service"), one or more CyclicBarrier objects, and a method that is called for every thread. This method contains a list of 'sub'methods and between these submethods, you can place CyclicBarriers. 

```
public class LionPenManager{

  -- method definitions of removeLions(), cleanPen() and addLions()

  public static void main(String[] args){

    ExecutorService service = null;
    try{
      service = Executors.newFixedThreadPool(4);
      var manager = new LionPenManager();
      var c1 = new CyclicBarrier(4);
      var c2 = new CyclicBarrier(4);
      for(int i=0;i<4;i++)
        service.submit(()-> manager.performTask(c1, c2));
    } finally {
      if(service!=null) service.shutdown();
    }
  }

  public void performTask(CyclicBarrier c1, CyclicBarrier c2) {

    try {
      removeLions();
      c1.await();
      cleanPen();
      c2.await();
      addLions();
    } catch(InterruptedException | BrokenBarrierException e) {

      -- handle exceptions
    }
  }
}
```

Note that CyclicBarrier throws exceptions that need to be caught somewhere. CyclicBarrier has overloaded constructors, you can use one with a second Runnable argument that runs upon completion.

Important is that the amount of threads created is at least as large as the CyclicBarrier limit value. If not, the thread will hang forever in a deadlock. In the code above, four threads are performing the tasks, and they can only proceed after four have completed the task (this is the 4 in the CyclicBarrier constructor).

If, for example, there are 15 threads (Executors.newFixedThreadPool(15)) and CyclicBarrier limit is set to 5 (new CyclicBarrier(5)), then the threads will proceed in groups of five. It is like waiting in a queue whereby each time a fifth person joins the queue, all five can pass. With fifteen persons you will get that moment of passing three times.










