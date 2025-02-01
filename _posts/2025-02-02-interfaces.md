## Interfaces 

It took me six week to figure out why 'Set' is rendered yellow and 'ArrayList' is rendered white in Intellij. Set is an interface, arraylist an object.

The idea of an interface as a contract that obligates you to include specific methods with specific arguments and specific return values already sunk in a few weeks ago. Using interfaces to separate implementation from distribution didn't yet, but with the [logger]() I created in my [Spaghetti project]() it is starting to dawn on me.

My pet project has a rudimentary logging system in which the distributed logger object is an interface and the implementation is an object. These are their codes:

```
package main.java.com.geertjankuip.logging;

public interface ActivityLogger {

    void logAction(String message);
}
```
```
package main.java.com.geertjankuip.logging;

import javax.swing.*;
import javax.swing.text.BadLocationException;
import javax.swing.text.DefaultStyledDocument;
import javax.swing.text.SimpleAttributeSet;
import javax.swing.text.StyleConstants;
import java.awt.*;


public class MyLogger implements ActivityLogger{

    DefaultStyledDocument doc;

    SimpleAttributeSet black = new SimpleAttributeSet();

    JTextPane target;
    StringBuilder totalLog = new StringBuilder("");

    public MyLogger(JTextPane target){

        this.target=target;
        this.doc = (DefaultStyledDocument) target.getStyledDocument();
    }

    @Override
    public void logAction(String message) {

        try {
            updateStyledDocument(message + "\n");
        } catch (BadLocationException e) {
            throw new RuntimeException(e);
        }

        target.repaint();
    }

    public void updateStyledDocument(String message) throws BadLocationException {

        StyleConstants.setFontFamily(black, "Consolas");
        StyleConstants.setFontSize(black, 16);
        StyleConstants.setLineSpacing(black, 1.0F);
        StyleConstants.setForeground(black, new Color(30,30,30));

        doc.insertString(doc.getLength(), message, black);
    }
}
```

Because I had not internalized the basic idea about this separation, I had simply implemented the MyLogger in all my classes instead of ActivityLogger (the interface). What I have understood now is that the receiving class sets the logger interface as an argument, while the sending class sends it as implemented object. I'm gonna try to think of more occasions where this would be beneficial.



