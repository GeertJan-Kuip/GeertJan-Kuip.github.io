## Debugging in Intellij part 3 

This is a topic I'm getting philosophical about. In previous posts I told about my encounters with the Intellij debugger and the difficulties using it.

Maybe the main problem with debugging is the information overload it generates. There's so much information that the debugger has to display variables using rather small fonts and collapsed collection values, as otherwise things won't fit on the screen. And as the values you are interested in (the value of expressions, or the n values that are generated in a loop) are not displayed automatically, you have to ask the debugger to display even more values so the screen becomes even more cluttered. It ends up with you looking at everything, instead of being focussed on the variable or expression you are most interested in.

In science, it is common practice to specify a hypothesis before you do the research. If your hypothesis states that the moon is made of cheese, you create an experiment that will fail if the moon isn't. Your effort is directed at just one aspect of the moon, which provides clarity to you and your peers. You won't simply go to the moon and have a look at everything. Philosophically, while it is possible to look at everything, you cannot see everything as seeing requires the presence of mental concepts.

To be fair, the Intellij debugger is equiped for this sort of focussed hypothesis testing. When setting a breakpoint you can write some very focussed evaluation code that will pass or fail the test. Unfortunately it does so in a very unarticulated way. Yes, you will notice if your test succeeds or fails, but the notification drowns in all the other information.

The main reason that programmers use System.out.println() might well be that they want simple and clear answers on one question instead of twenty. System.out.println() implicates a hypothesis (this variable might be null). You do a small experiment (run the program) and get a direct answer to the initial question (this variable is indeed null). It might not be the most efficient way to find bugs but at least you don't drown in information.

Anyway. I might try to create some own tool, one that is less primitive than System.out but allows for unambiguous hypothesis testing. I'm thinking of a static class with a very short name ('K') and methods with very short names line 'p'. Adding the line K.p(variable name) will automatically result in a sort of System.out.println, with better printing of collections, objects, resultsets and what more, and the printing can be done to a textfile or whatever works best. If you place K.p(variable name) in a loop, it should limit itself to a certain amount of content as to prevent overload. There should also be some easy shortcut to disable the working of K.

Okay, this might not work at all but the topic still bothers me. There will be a part 4 in this series.
