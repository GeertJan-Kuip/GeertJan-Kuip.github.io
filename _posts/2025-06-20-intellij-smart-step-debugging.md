# Intellij smart step debugging

While trying to understand Maven 4.0.x, I used Intellij's debugger to guide me to the code. Basically you set a breakpoint at the first line in the main method and from there on you use either 'Step over' or 'step into' to navigate through the code. Upon debugging like this, it dawned upon me that I was traversing the code in a way different from the way it was executed during runtime. Here is an example of code at the start of the program:

```
        try (Invoker invoker = createInvoker()) {
            return invoker.invoke(createParser()
                    .parseInvocation(createParserRequestBuilder(args)
                            .stdIn(stdIn)
                            .stdOut(stdOut)
                            .stdErr(stdErr)
                            .embedded(embedded)
                            .build()));
        }
```

During runtime, the createInvoker() method is called first, and it is followed by a streak of methods that are to be found elsewhere in the code base. If the createInvoker() things are done, the createParserRequestBuilder(args) is executed, not the invoke(), createParser() or .parseInvocation() methods. Those can only be executed when the argument of .parseInvocation() has been generated. Anyhow, code is not executed token by token.

## Smart step debugging

Smart step debugging is the default of the debugger and it lets you select which method you want to step into if you press F7/Step Into. You can select everything from .invoke() till . build() by clicking on the screen, and as a default it selects .invoke() for you. Here is the problem: it is illogical to have .invoke() as default, as this is the last one that will be executed during runtime. By using defaults, you follow a route that differs from the way the code is executed. Sometimes that is exactly what you want, but in my case, I did not only want to go to the invoke() method, but also see which argument is being used.

## Turn it of

In Settings->Buid, Execution, Deployment->Stepping there is an option (the first) 'Always do smart step into'. Turn it of. By disabling it, you can traverse the code exactly the way that it is being traversed during runtime.