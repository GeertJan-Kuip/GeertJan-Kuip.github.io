## Debugging in Intellij part 2 

I learned a bit more about the Intellij debugger, by using it and by watching an Intellij YouTube video in which one of its developers explained features. I have internalized a few things:

- you can set non-suspending breakpoints and ask the debugger to print log messages in the console each time it encounters that breakpoint. It is an alternative to System.out.println.
- To get the value of an expression, you can alt-click on it.
- Chains of methods in forEach and stream-like constructs can be be inspected with a rather nice tool where you see all variables passing the stream and the way they are modified by each method.

But still it doesn't work as great as I hoped it would work, probably because I'm still not virtuous with it. What I want it to do is for example:

- traverse the code backwards, instead of forward, starting at the point where the exception has occurred.
- a default setting in which the debugger shows none of the variables values. There is an overload of information now and simple variables get preference above complex expressions.
- the line where you put the breakpoint should be included in code execution, especially if it is the last line of a code block.

The debugger is great, learns you a lot about the way code is executed. It is also an expert tool that only works if you master it well. I'm not there yet.
