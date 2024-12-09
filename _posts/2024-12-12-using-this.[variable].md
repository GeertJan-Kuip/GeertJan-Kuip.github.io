## Using this.\<variablename\>

In a previous post I lauded the use of classes, not only to better organize code but also to get rid of the mess in your head that emerges when your mind has to keep track of countless variables getting passed from function to function.

By encapsulating my problem in a class and by putting the class in its own file, things got way better. But I still got one thing wrong. Within the class, I was still passing variable around from function to function while I shouldn't.

By declaring all class variables in the constructor function (without even giving them a value), I gave myself the comfort of a comprehensive list of all variables that can be called anywhere in the class body by using 'this'. No more need to pass them around in method brackets. So much better.