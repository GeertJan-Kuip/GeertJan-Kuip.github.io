## Exception handling

Preparing for 1Z0-819 I'm learning about exception handling. I read the [Java tutorial](https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html) and fiddled around in Intellij. This is what I learned:

- All exception (and error) classes have Throwable as ancestor. Exception is the parent of RunTimeException. Only subclasses of RunTimeException are counted as unchecked exceptions.
- StackTraceElement is a class whose instances contain variables file, class, method and line. StackTraceElement plays an important role in Throwable, the stacktrace is an array of StackTraceElements.
- e.printStackTrace() prints a list of StackTraceElements to the terminal. e.getSTackTrace() returns an array of StackTraceElements that you can use to do your own custom reporting, something like:
```
try {
    throw new SQLException();
}catch (SQLException e){

    StackTraceElement ste[] = e.getStackTrace();
    int i = 0;
    System.out.printf("\u001B[34m%s - Class: %s - method: %s - line:%s%n\u001B[0m",
            ste[i].getFileName(), ste[i].getClassName(), ste[i].getMethodName(),
            ste[i].getLineNumber());

}
```
- Subclasses of Throwable have very little code in them, and refer to methods in Throwable via the 'super' keyword.
- Throwable has 4 public constructors and one protected:
```
Throwable();
Throwable(String message);
Throwable(String message, Throwable cause);
Throwable(Throwable cause);
(protected) Throwable(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace);
```
The latter one is accessible by all subclasses, not by clients.
- Throwable hase a 'cause' variable that facilitates exception chaining. Exception chaining helps to create better log content.
- A chained exception is created by throwing a new exception in the catch block with the 'caught' exception as 'cause' parameter.
- Some of the relevant methods in Throwable, apart from its constructors and the print methods, are:
```
getCause();
getStackTrace();
getSuppressed();
initCause(Throwable cause);
```
- Exceptions must be caught or thrown up the call stack. It is often better to catch late as it might allow you to do something about the problem instead of just reporting it.
- A try-catch-final needs a catch, a final, or both. Multiple catch blocks are allowed.
- When having multiple catch statements, you should start with the least generic. If a catch statement can't be reached because it is preceeded by a catch statement catching a parent exception class, code won't compile.
- If both the try and the catch block result in an exception, the exception originating in the try block will be suppressed.
- To get information about the suppressed exception of the try block, do e.getSuppressed() in the catch block.
- If in a subclass you override a method that has a throw in its signature, the throw in the subclass should be equally or more narrow than that of the parent class. You cannot pick a more generic exceptionclass.

Try-with resources
- A try-with-resources can have catch and final blocks but not necessarily so.
- In a try-with-resources the try-with-resources statements are placed within parentheses just after try, separated by semicolons. Semicolon after last statement is optional. This statement section needs contain the opening of the resources. The try block {} comes after, containing all the code that requires the content of those resources.
- If an exception occurs in a try-with-resources, the resources are closed first and only then the catch and finally blocks are executed.
- Resources in the try-with-resources statement are closed in the reverse order in which they were declared.
- If a try-with-resources fails, both in the statement(s) and the block, the exception from the statement(s) is suppressed and the exception from the block is thrown.
- Try-with-resources can very well be used when writing and reading databases. It guarantees that connections, resultsets and statements will be closed.
- Try-with-resources closes all resources that implement the java.lang.AutoCloseable interface or its subinterfaces, such as java.io.Closeable.
- A situation can occur in which the Java garbage collector reclaims a resource before it is being closed. The operation system will still think that the resource is being used so will not close it itself. Actually the resource can't be closed anymore as the GC removed the information needed to close it. This is a memory leak. Try-with-resources is the best way to prevent it.
- AutoCloseable and Closeable throw different exceptions, respectively Exception and IOException. The Exception of AutoCloseable is too generic and it is recommended to override it in implementing classes.
- From Java docs AutoCloseable: 'Note that unlike the close method of Closeable, this close method is not required to be idempotent. In other words, calling this close method more than once may have some visible side effect, unlike Closeable.close which is required to have no effect if called more than once. However, implementers of this interface are strongly encouraged to make their close methods idempotent.'
- It is not recommended to close a resource twice but it will work and it will compile. It can happen if you open the resource in a try-with-resources statement and afterwards close it manually, for example in a finally block.

- You can throw an exception up until main, if main throws it as well the program will terminate.
- A stacktrace is automatically printed in the terminal if a throw is not caught or handled.
- Exception handling in big systems consisting of different parts with different owners is a controversial topic.
- The final block will always execute, unless the JVM terminates. Before the introduction of try-with-resources, the finally block was used to close resources.
- A finally block can throw an exception, it is therefore possible that not the whole finally block will be executed.
- e.printStackTrace() prints to the terminal in red color. That looks scary, as if many things have gone wrong, but it is mainly optics. If you do some other kind of writing to the terminal instead of e.printStackTrace the color might not be red but the problem is exactly the same.
- You can adjust the print color in the terminal to blue by preceeding the string to print with "\u001B[34m". End the string with "\u001B[0m" to set it back to default color. Other colors can be set as well, it is the two digits before the m that indicate the color.
- A catch statement can contain more than one exception object type. Separate them by a vertical bar.
```
catch (IOException|ArrayOutOfBoundsException e) {}
```
- This construction, with two exception objects, makes e final.
- If two or more ExceptionClasses are declared in catch statement, one cannot be a subclass of another. If so, compilation fails.





