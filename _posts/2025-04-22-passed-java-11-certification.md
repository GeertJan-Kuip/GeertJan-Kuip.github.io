## Passed Java 11 certification

This morning I successfully passed the Oracle Java SE 11 Developer 1Z0-819 exam. The required score was 68% and that was what I exactly had. The badge is on my [LinkedIn](https://www.linkedin.com/in/geert-jan-kuip-95387630).

It is a [little more than four months ago](https://geertjan-kuip.github.io/2024/12/17/JAVA!.html) that I started learning Java, and this is what Oracle think the badge is worth:

_Candidates who hold this certification have demonstrated proficiency in Java (Standard Edition) software development recognized by a wide range of world-wide industries. They have also exhibited thorough and broad knowledge of the Java programming language, coding practices and utilization of new features incorporated into Java SE 11. By passing the required exam, a certified individual proves tremendous fluency in Java SE and acquisition of the valuable professional skills required to be a Java software developer. This includes a deep understanding of object-orientation, functional programming through lambda expressions and streams, and modularity._

### A rewarding experience

Learning for this exam, which I started in february, was in fact a joy and to me all the topics seemed relevant. I indeed feel very thoroughly educated in the basics of Java. Inheritance, polymorphism, scope, functional programming, generics, concurrency, exceptions, modular applications, i/O and NIO.2, JDBC and security were all in it. I have learned, more than I would otherwise would have, to think as a compiler.

### Learning materials

The most important learning material I used was the book "Oracle Certified Professional Java SE 11 Developer Complete Study Guide Exam 1Z0-815, 1Z0-816, and Exam 1Z0-817." The book is a few years old and the title might seem to be misleading but Oracle has combined some previous exams at different levels into one new exam that covers them all. The authors Jeanne Boyarsky and Scott Selikoff have done a terrific job. The text is crystal clear, the review questions are difficult enough to keep you humble and in my experience the exam was completely in sync with the book. As far as I can recall there were no questions on the exam that the book hadn't covered.

I also bought their other book, "Oracle Certified Professional Java SE 11 Developer Practice Tests." This book has tons of test questions on all topics, including 4 practice exams covering everything in 50 questions, and it is just good, although I spent much more time in the study guide. 

In the week before the exam I also bought the [Enthuware](https://enthuware.com/) mock tests, you pay around 12 dollar for access to around 1000 test questions that you take online. The good thing here was that the questions were challenging, the explanations were well formulated and helpful, and the experience (doing a set of 50 questions covering the whole spectrum with the possibility to mark them, revisit them etc) very much emulated the real test. 

The last thing I had were online tests from [Udemy](https://www.udemy.com/course/java-se-11-developer-1z0-819-ocp-course-part-1/?couponCode=ST8MT220425G1). I did one or two tests of them, a bit similar to Enthuware, with slightly easier questions. In the week up to the exam I had scores of around 55% on the mock exams of Enthuware and those of Jeanne and Scott, and a 66% on a Udemy exam. Nevertheless, for a price of 10 or 15 euros it is great value. You might also argue that the mock exams were I got 55% were slightly too difficult (although I don't think so).

### Source code and the JLS

The best investment I did, apart from thoroughly reading the Complete Study Guide and creating all sorts of coding examples from it, was reading the Java code in the standard library and reading parts of the [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se11/html/index.html). 

#### Standard library

It is hard to obtain a good mental representation of quasi-abstract things like 'the Object class,' the 'Collection interface' or 'Throwable'. What helps is to study the .java files from the JDK. The moment you have seen with your own eyes that all the basic content of Exception classes can be found in Throwable gives you immediately more feeling for the inheritance structure of exceptions. Viewing class 'Object' for yourself will keep you reminded all the time that a specific set of methods is implicitly to be found in every class. It also makes you understand what the logic is of 'synchronized(someObject)'. Learning functional interfaces becomes much easier when you have seen how incredibly simple the implementation is, and the mysterious 'System.out.println' becomes sort of obvious once you know where to find the class System and once you realize that 'out' is a static field of type PrintStream, and that print, printf and println are instance methods from the PrintStream class. 

All in all, studying implementations as found in the standard library helps to both understand and remember things better. On top of that, there is a personal touch to it as you can directly see who is the [author of the code](https://geertjan-kuip.github.io/2025/01/04/arthur-van-hoff.html) , which makes remembering even easier.

#### Java Language Specification

I probably should have spent even more time reading the JLS. It is tremendously helpful in learning to think as a compiler. A topic where carefull study of the JLS helped me was [scope](https://geertjan-kuip.github.io/2025/04/03/scope.html), to be found in [paragraph 6.3](https://docs.oracle.com/javase/specs/jls/se11/html/jls-6.html#jls-6.3). The alternative for reading the JLS is reading books, which are by definition less precise or less complete, or trying to figure out the rules of the compiler just by experiment. While this is a good excercise, it is probably impossible to infer the working of the compiler just by doing code experiments. The JLS is simply a gem.

### What's next

I want a Java job and I have some first urgent new learning objectives. About the latter, there's the observation that preparing for the exam learned me a lot of things theoretically which I have actually not tried out, or just very limited, in practice. To name a few:

- compiling source files, manually creating appropriate directory trees.
- create working JARs of packages and modules from command line.
- create a working application with modules, also using services.
- using annotations in a meaningful way.

These things all play out on a level above that of class files or even packages. What I notice is that it it easy and convenient to fiddle around with simple classes and methods in the safe environment of Intellij (or even some simple sandbox as the [Java playground](https://dev.java/playground/) but that this safe environment is not very conducive to learning to manage large applications. So that's the next objective.

About the Java job: I hope I can get in touch with team leads or other managers etc in the Java world to learn more about what it is that I can do as a starting Java developer. At least I have a badge to show now.










