## Deadlock, starvation, livelock and race conditions

Part of [chapter 18](https://www.amazon.com/gp/product/B08DF4R2V9/ref=ppx_yo_dt_b_d_asin_title_351_o00?ie=UTF8&psc=1) on concurrency is a paragraph called 'Identifying Threading Problems.' Using all the classes to make thread safe code might not be enough to avoid problems in multithreaded code. Ironically it is possible to run into new problems just because of those classes. 

### Liveliness

Liveliness is the ability of an application to execute in a timely manner. Liveliness problems are those in which the application becomes unresponsive or in some kind of "stuck" state. The book distinguishes three types of liveliness issues that should be learned namely deadlock, starvation and livelock.

#### Deadlock

Deadlock is when two threads are blocked forever because they need to wait for each other. The book presents a code example of two foxes who cannot get their drink/food because the other fox is occupying the drink/food place. 

#### Starvation

A thread is perpetually denied access to locks or shared resource. Although the thread is active, it can do nothing as other threads take the resources the starving thread is after.

#### Livelock

Two threads are in a never ending loop of responding to each other, like two lovers on the phone each insisting that the other should hang up first.

### Managing race condition

"A race condition is an undesirable result that occurs when two tasks, which should be completed sequentially, are completed at the same time." Think of two people creating an account with the same user name at the same moment. The best solution is to give one user precedence to the other, even if they arrive at exactly the same moment.

So far. This is the fourth on concurrency (a short one). One more on this topic to follow.

