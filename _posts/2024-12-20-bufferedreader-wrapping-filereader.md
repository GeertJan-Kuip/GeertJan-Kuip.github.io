## BufferedReader wrapping FileReader

The thing I want to build requires reading textfiles, so I need to get familiar with the java.util.io package. Web search learned me that the right way to import text from a text file is:

```BufferedReader importedText = new BufferedReader(new FileReader("sometext.txt"));```

In a previous life I would have happily copied this code, rushing for some fancy result. Not anymore, now I wonder why:

```FileReader importedText = new FilerReader(new BufferedReader("sometext.txt"));```

isn't the way to go. BufferReader is used to prevent overly accessing the external source, which should mean that it should be BufferedReader and not FileReader which gets primary access to the something.txt file.

I'm puzzled, in the Java documentation I'm trying to figure out the class hierarchy and I have noticed that in Intellij I can read the source code of all java classes.

While doing so, I found that the .nio package is way more interesting than the .io package. Let's get into that rabbit hole as well.