## Creating a logger 

In my pet project, a desktop application that can analyze a Java codebase, you need to select a folder upon which the program will start to tokenize and analyze the .java files in it.

It seemed appropriate to have some sort of logging mechanism in it that records and displays all the steps in this process. I found a [post on Stackoverflow](https://stackoverflow.com/questions/4297729/sending-messages-to-a-swing-jtextarea-from-different-places) that provided some example code and used it: 

```
package com.example.logging;
public interface ActivityLogger {
    void logAction(String message);
}
```
```
public class FileLoader {

    private ActivityLogger logger;
    public FileLoader(ActivityLogger logger){
        this.logger = logger;
    }

    public void loadFile(){
        // load stuff from file
        logger.logAction("File loaded successfully");
    }

}
```
```
public class TextComponentLogger implements ActivityLogger{
    private final JTextComponent target;
    public TextComponentLogger(JTextComponent target) {
        this.target = target;
    }

    public void logAction(final String message){
        SwingUtilities.invokeLater(new Runnable(){
            @Override
            public void run() {
                target.setText(String.format("%s%s%n", 
                                             target.getText(),
                                             message));
            }
        });
    }
}
// Usage:
JTextArea logView = new JTextArea();
TextComponentLogger logger = new TextComponentLogger(logView);
FileLoader fileLoader = new FileLoader(logger);
fileLoader.loadFile();
```

The idea is that only one Logger object is created (you can choose to make it a simpleton but it's no problem if you don't) and that this one object is distributed to all classes that have something to log. It uses 'constructor dependency injection' for it, which means that any class using the logger will have the logger as one of its constructor parameters. According to what I have read so far it's good practice.

One thing: I haven't managed yet to let Java Swing do the log output one line at a time. Somehow it buffers all of the log messages and then dumps them all at once. That doesn't look cool, hope I will fix it.
