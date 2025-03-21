## Mutable result container

While practicing streams and functional programming I challenged myself to create some IntStream and to find the highest value in it, without using the dedicated oparators max() and summaryStatistics(), and without mapping the stream to an object stream by using boxed() or mapToObj(). The only one I allowed myself to use was collect().

Spoiler alert: without perplexity.ai I would not have found the solution.

Collect() in IntStream has only one signature and it lacks the ability to work with a Collector object. You cannot use collect(Collectors.<some fancy method>). All there is is this signature:

<R> R **_collect_**(Supplier<R> supplier, ObjIntConsumer<R> accumulator, BiConsumer<R,R> combiner)

Personally I find it hard to figure out what such signatures, with their generics and in this case an obscure functional interface 'ObjIntConsumer', exactly represent. But I studied it a bit and I understand it a bit better. But let's show the solution first:

```
int[] maxValue = IntStream.range(0,20)
        .parallel()
        .collect(
                ()->new int[1], // supplier of mutable result container
                (res, x)->{     // accumulator
                    res[0] = x>res[0] ? x : res[0];
                },
                (a,b)-> a[0] = a[0]>b[0] ? a[0] : b[0]   // combiner
        );

System.out.println(maxValue[0]); //19
```` 

Let's break it down:
- an IntStream of 20 subsequent int primitives is created
- they are processed in multiple threads
- the collect method consists of three functional interfaces with the roles of supplier, accumulator and combiner

If you want to use the collect method in an IntStream (or a LongSTream or a DoubleStream) you need to use this implementation, you cannot create a Constructor object like you would do in Stream.

### The supplier

The supplier creates the object to store updates in, which must be a **mutable result container**. This means that it cannot be a String or a primitive (both are immutable). It must be an object that you can adjust without have to create a new copy of it. This accumulator will be the return value of the collect method and during the process it will be continuously adjusted, at least as the stream is not empty.

My initial idea of using an int primitive as accumulator was thus a bad idea. Using some collection was also a bad idea, as collections do not hold primitives. I would have to cast int to Integer which is what I wanted to prevent. The solution to use an array (perplexity proposed it) was the hack I needed. It doesn't get more basic than this.

An alternative would be to create some custom object with a int field that can be get and set. In the code below I dod so, I created a class IntWrapper that holds one int value.

```
IntWrapper maxValue = IntStream.range(0,20)
    .parallel()
    .collect(
            ()->new IntWrapper(-2_147_483_648),   	// supplier of mutable result container
            (intWrapper, x)->{				// accumulator
                intWrapper.setIntPrimitive(Math.max(intWrapper.getIntPrimitive(), x));
            },
            (a,b)-> a.setIntPrimitive(Math.max(a.getIntPrimitive(), b.getIntPrimitive()))   // combiner
    );

System.out.println(maxValue.getIntPrimitive());  // 19
```

Instead of working with a ternary operator I used Math.max to get the highest of two values, I could have done that in the previous version as well. I set the inital value of the int in IntWrapper to the lowest possible value for an int, as so the code works as well with the lowest possible values. This was suggested by Perplexity. Perplexity used Integer.MIN_VALUE, which is actually better practice than -2_147_483_648.

### The accumulator

According to the specification, the second argument of the count function has type ObjIntConsumer<R> accumulator. This functional interface is a specific sort of BiConsumer that takes an object of type R and an int as parameters. This object type R is actually the mutable result container that was created in step one. I used an int[] instance and an IntWrapper instance. 

What I found confusing is that a BiConsumer is used and not a BiFunction. I expected that a return value was needed and that that return value would somehow be added to the mutable result container. Now that I thought about the nature of the mutable result container it has become clearer to me that a BiConsumer is more appropriate, as you need specific code to add to the container. There is no generic 'add to the total result' method that would justify using a return value.

### Combiner

The combiner only has a role when parallel threads are running. It states how partial results are to be combined, whereby the first argument (same type as the mutable result container) is the aggregating object and the second argument is the object that is to be combined to the first. In this case, finding the highest value, the combiner is very similar to the accumulator. There are some guiding principles for when a combiner is correct but I cannot recall them now.

### Lessons

There is some arbitrariness about the way Java methods are implemented, therefore there are a lot of specific things to learn or even to memorize. What helps to understand this collect method a bit more deeply is the following:

- The mutable result container is the main element
- It cannot be something immutable, not a string and not a primitive
- The accumulator must be a BiConsumer<U,T>, because there must be two input values, namely the mutable result container and the element that is streamed
- BiConsumer will mostly have two type parameters, although not necessarily. In this case ObjIntConsumer<R> suggest there is one parameter type but of course there are two (R and int)
- The accumulator can not be a BiFunction, this would not give you the leeway to make a proper accumulation
- The combiner must be a BiConsumer<R>, the elements to be combined are by definition of the same type and that same type is the output type of collect







  


