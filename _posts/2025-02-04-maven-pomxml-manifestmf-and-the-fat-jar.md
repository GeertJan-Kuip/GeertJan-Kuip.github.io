## Maven, pom.xml, MANIFEST.MF and the fat jar 

I started my pet project using the default Intellij setup, foregoing Maven. Last friday I decided to convert it into a Maven project and to upload it to GitHub so my versioning would be correct and others would be able to see it.

By reading about it I was made aware of the blessings of Maven, the problems it solves with complex builds and dependencies, the structure of pom.xml, the required folder structure (convention over configuration) and the importance of manifest.mf and the place in the directory tree where it should reside.

What I ran into after letting Maven do its job was a sort of limbo where traces of the old directory structure resided next to the Maven structure. By accident I had compiled some module from my code, which I deleted, and I ended up with I think was an unconnected Main function, probably because the Main file ended up in a wrong place in the directory tree, which resulted in package imports that did only work after I added the prefix main.java. to them.

Anyway, it is the sort of process in which you decide to make a backup of your project folder in case you will loose everything.

Nevertheless, things kind of work again. I even created a working JAR file in which the dependencies (the driver for the SQLite database) were included. Unfortunately I could only get that result by using the Intellij build tool, because Maven refused to do so. I learned that Maven needs the shade plugin to make these fat jars so I put that one in pom.xml, and now it does make a fat jar but it won't execute. I might have to remove all the prefixes, adjust manifest.mf and/or check if there is no double code in the project.

I like programming and I'm getting better at it. But I need to get better at configuration if I want to make things that others can see and use.
