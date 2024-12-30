## Passing object references

I needed a simple interface so I discovered Java Swing. JavaFX makes cooler interfaces but needs separate installation, so I sticked to the older solution. 

While coding the interface I ran into the problem of how to make adjustments to the interface, represented by the GUI class I created, fromout my MyActions class. More specifically I wanted the text in a textPane element to change upon pressing a button. This problem is rather typical for object-oriented programming.

My first solution was to make the GUI textPane variable static, meaning that I would be able to alter the content of the textPane by writing

```GUI.myTextPane.setText("new text");```

The internet strongly disagreed with this solution, arguing that once you make GUI elements static, it is problematic to ever scale your project up to something in which you can run multiple instances of your GUI.

The solution I ended up for now is including the GUI instance as parameter of those methods of the Action class that are meant to alter the contents of GUI elements. When calling this methods fromout the GUI class the instance is passed by using keyword 'this'. It works but I'm not 100% happy with having to pass many things around. It reminds me of my older, more chaotic functional style of programming.