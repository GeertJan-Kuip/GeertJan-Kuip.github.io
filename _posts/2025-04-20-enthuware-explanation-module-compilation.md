## Enthuware explanation module compilation

[Enthuware](https://enthuware.com/) has good explanations for why a certain answer is right or wrong. I copied this one, just to have it at hand. It is about module compilation:

### On compiling

For compiling a Java class that is part of a module, you need to remember the following five command line options:

1. --module-source-path: This option is used to specify the location of the module source files. It should point to the parent directory of the directory where module-info.java of the module is stored. For example, if your module name is moduleA, then the module-info.java for this module would be in moduleA directory and if moduleA directory exists in src directory, then --module-source-path should contain the src directory i.e. --module-source-path src

If moduleA depends on another module named moduleB, and if moduleB directory exists in src2 directory, you can add this directory in --module-source-path as well i.e. --module-source-path src;src2. javac will compile the required files of src2 as well if the source code of moduleB is organized under src2 correctly.


2. -d: This option is required when you use the --module-source-path option. It is used to specify the output directory. This is the directory where javac will generate the module's package driven directory structure and the class files for the sources. For example, if you specify out as the output directory, javac will create a directory under out with the same name as the name of the module and will create class files with appropriate package driven directory structure under that directory.


3. --module or -m: This option is used when you want to compile all the source files of a particular module. This option is helpful when you want to compile all the files at once without listing any of the source files of a module individually in the command.
For example, if you have two java files in moduleA, stored under moduleA\a\A1.java and moduleA\a\A2.java, you can compile both of them at the same time using the command: javac --module-source-path src -d out --module moduleA
Javac will find out all the java source files under moduleA and compile all of them. It will create the class files under the output directory specified in -d option i.e. out. Thus, the out directory will now have two class files - moduleA/a/A1.class and moduleA/a/A2.class.


4. --module-path or -p: This option specifies the location(s) of any other module upon which the module to be compiled depends and is very versatile. You can specify the exploded module directories, directories containing modular jars, or specific modular jars here. For example, if you want to compile moduleA and it depends on another module named abc.util packaged as utils.jar located in thirdpartymodules directory then your module-path can be thirdpartymodules or thirdpartymodules/utils.jar. That both the following two commands will work:

javac --module-source-path src --module-path thirdpartymodules -d out --module moduleA
and
javac --module-source-path src --module-path thirdpartymodules/utils.jar -d out --module moduleA

Note:If your module depends on a non-modular third party jar, you need to do two things -
Put that third party jar in --module-path.
Putting a non-modular jar in --module-path causes that jar to be loaded as an "automatic module". The name of this module is assumed to be same as the name of the jar minus any version numbers. For example, if you put mysql-driver-6.0.jar in --module-path, it will be loaded as an automatic module with name mysql.driver. Name derivation is explained in detail in java.lang.module.ModuleFinder JavaDoc but for the exam, just remember that hyphens are converted into dots and the version number and extension part is removed.
It is also possible for a non-modular jar to specify its module name using Automatic-Module-Name: <module name> entry to the jar's MANIFEST.MF.
Add a requires <module-name>; clause in module-info of your module.

5. -classpath: This option is used for compilation of non-modular code. If you are compiling regular non-modular code but that code depends on some classes, then you can put those classes or jars on the classpath using -classpath option.
Note: This option is not helpful for compilation of modular code because classes of a modular cannot see classes on classpath. Modular code can only see other modular code. That is why, non-modular classes have to be converted into "automatic modules" and put on --module-path as explained above.

### On launching a modular application

Top Down Approach for modularzing an application

While modularizing an app in a top-down approach, you need to remember the following points -

1. Any jar file can be converted into an automatic module by simply putting that jar on the module-path instead of the classpath. Java automatically derives the name of this module from the name of the jar file.

2. Any jar that is put on classpath (instead of module-path) is loaded as a part of the "unnamed" module.

3. An explicitly named module (which means, a module that has an explicitly defined name in its module-info.java file) can specify dependency on an automatic module just like it does for any other module i.e. by adding a requires <module-name>; clause in its module info but it cannot do so for the unnamed module because there is no way to write a requires clause without a name.  In other words, a named module can access classes present in an automatic module but not in the unnamed module.

4. Automatic modules are given access to classes in the unnamed module (even though there is no explicitly defined module-info and requires clause in an automatic module). In other words, a class from an automatic module will be able to read a class in the unnamed module without doing anything special.

5. An automatic module exports all its packages and is allowed to read all packages exported by other modules. Thus, an automatic module can access: all packages of all other automatic modules + all packages exported by all explicitly named modules + all packages of the unnamed module (i.e. classes loaded from the classpath).

Thus, if your application jar A directly uses a class from another jar B, then you would have to convert B into a module (either named or automatic). If B uses another jar C, then you can leave C on the class path if B hasn't yet been migrated into a named module. Otherwise, you would have to convert C into an automatic module as well.

Note:
There are two possible ways for an automatic module to get its name:
1. When an Automatic-Module-Name entry is available in the manifest, its value is the name of the automatic module.
2. Otherwise, a name is derived from the JAR filename (see the ModuleFinder JavaDoc for the derivation algorithm) - Basically, hyphens are converted into dots and the version number part is ignored. So, for example, if you put mysql-connector-java-8.0.11.jar on module path, its module name would be mysql.connector.java