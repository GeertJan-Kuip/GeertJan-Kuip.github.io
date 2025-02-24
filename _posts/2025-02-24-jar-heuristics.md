## Jar heuristics


I want to get better with the command line but the previous blog post was a bit messy. In this blog I want to keep the scope smaller, it is just about creating the simplest jar files. No modules, only packages, with methods to create the right MANIFEST.MF file. Below some summarizing of what I learned, mainly from the [Java tutorials](https://docs.oracle.com/javase/tutorial/deployment/jar/index.html).


### The simple jar command

To create a simple jar file you compile your code (or part of it) in some folder. Then you can use ```jar -cf myApp.jar *```. A jar file that contains all the files in the current directoryin the correct directory tree, is placed in the current directory. In the jar file a META-INF folder is placed at the root with in it a MANIFEST.MF file. This file contains some information, but not the name of the class with main in it. If the jar is meant as a working program that can be run, it won't because of that.


### Flags

#### Essential flags

The -cf part of the simple jar command contains multiple characters, some more relevant than others. Their order matters, as it has to match the order of the arguments that follow it.

The first character is c (create), t (table) or x (extract). To create a jar file use c, to view the contents of a jar file use t, and to extract folders or files from a jar file use x.

The f character at the end stands for file. It means that a file is created and not some other type of output. You will always use it.


#### Non-essential flags

You can place a v between the c and f which stands for verbose. Verbose does not affect the end result but results in more log output during the making of the jar file. If you use t instead of c and add v, you will get extra info about the contents of the jar file.

Another flag is 0, which results in an uncompressed jar file. A jar file is created, but the content is not zipped. 


#### The m flag

The simple command ```jar -cf myApp.jar *``` creates a jar file but the MANIFEST.MF in it will not have a entrypoint in it. Therefore the jar file will not run, it is just an archive. Adding the m to the list of flags enables you to create a txt file with headers in it. Those headers will be added to the manifest file. If you create a txt file named sometitle.txt in the root folder and write a header-line ```Main-Class: someClass``` in it, you can include that specific header in the manifest file.

To do so, run ```jar -cfm myApp.jar sometitle.txt *``` in the command line. All headers that are in the sometitle.txt file are added to the MANIFEST.MF file. The jar file can be run now because the JVM can find the entrypoint.

Few notes:
- You need to write the FQN (Fully Qualified Name) of the entry class in the header, with dots separating the words.
- In your textfile, make sure that a line break is added after the header. Without it the header won't be read.
- The m need to be after the f (cfm, not cmf) as the order of the flags must correspond with the order of the arguments.


#### The M flag 

You can add M to skip the creation of a MANIFEST.MF file. This option can be useful if you already have a complete MANIFEST.MF file, in a META-INF folder, in the root folder. 


#### The e flag

This is a very useful one. Adding e enables you to add a FQN entrypoint in the command, that will be added to the MANIFEST.MF file. It is the easiest way to make a jar file that can be run. It will be like ```jar -cfe myApp.jar mypackage.ClassWithMain *```. A header ```Main-Class: mypackage.ClassWithMain``` is added. Note that the FQN (Fully Qualified Name, the one with the dots) needs to be provided.


### Specifying the input files

The jar command ends with a list of folders and files that will be included in the jar file. You separate them by a space if it is more than one. In the previous examples we used wildcard * to indicate that all contents of the root had to be included, but you can be more specific.


#### The -C flag

The -C flag changes the directory during the execution of the command. If, for example, you type ```jar -cvf myApp.jar -C images . -C audio .```, the contents of both the folder 'images' and the folder 'audio' will be added to the jar, without the folder. The dot indicates that the whole folder content is to be added. 


#### Wildcard *

Using * as last argument means that all the content of the current directory is added to the jar, following the same directory structure. The wildcard can be used in many ways. When, for example, you write package1/*.class as last argument, package1 is added with all class files in it. 


#### Table and extract

When using t instead of c jar will show the contents of the jar file you specify. Adding v as a flag shows additional information, such as the creation dates of files and folders. Typical use: ```jar -tvf myApp.jar```. 

If you want to extract the contents of a jar file to the current directory, type ```jar -xf myApp.jar```. The extracted files, including their directory structure, will be placed in the current directory, existing files with the same names will be overwritten. You can make a selection of files/folders to extract by adding their names to the command as last arguments.


### Endnote

In this blog I omitted some important topics, namely:

- jar files with modules
- jar files that refer to files outside of the jar file
- nested jar files
- manifest attributes other than Main-Class
- sealing packages
- verifying and signing jar files

Those are for later.






