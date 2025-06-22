# Plugins in IntelliJ

Two months ago I explored the possibilities of creating my own plugin in IntelliJ, left it, and two days again I came back to it. The problem I want to solve is how to explore complex code faster and easier, and as IntelliJ has an SDK that allows for all sorts of extensions, there might be some opportunity. The only problem is that the SDK is so extensive that it is hard to grasp the full extent of the toolbox.

## The Program Structure Interface

The main element I'm interested in is the PSI, Program Structure Interface. IntellijJ IDEA parses files and turns code files into a tree of elements like this:

- A file becomes a PsiFile object, containing a hierarchy of PSI nodes
- Each method, class, field or parameter is a PsiElement
- You can resolve method calls to declarations using PsiReference.resolve()

When you navigate through code in IntelliJ it is the PSI that lets you traverse chains of method calls, find either the parent or child classes of classes or interfaces, and find usages and overrides of methods.

According to ChatGPT, there exists a PsiViewer plugin that lets you see the PSI tree of any file.

### Relevant Psi classes

ChatGPT also provided this table with some of the relevant Psi classes:

|Class|Description|
|----|----|
|PsiElement|Base interface for any code element|
|PsiMethod|Represents a method declaration|
|PsiReferenceExpression|Represents a method/field/variable reference|
|PsiFile|Represents a file in the PSI tree|
|PsiClass|Represents a class or interface|
|JavaPsiFacade|Entry point for querying PSI structures|
|PsiReference|Interface for resolving references to declarations|

### AnAction

In most of the samples the plugin is written as a Java class extending the abstract AnAction class. [Here](https://plugins.jetbrains.com/docs/intellij/creating-actions-tutorial.html) it is described. You must implement `actionPerformed()`, `getActionUpdateThread()` and eventually `update`.

### Registering an action

To make the action available via the interface it must be registered. This is done by adding an action section to the plugin.xml file that sits in the folder `\src\main\resources\META-INF`:

```
<actions>
  <action
      id="org.intellij.sdk.action.PopupDialogAction"
      class="org.intellij.sdk.action.PopupDialogAction"
      text="Popup Dialog Action"
      description="SDK action example">
    <add-to-group group-id="ToolsMenu" anchor="first"/>
  </action>
</actions>
```

### Gradle and general configuration

The cumbersome part of the plugin development process has to do with the configuration that needs to be done in two files, the `plugin.xml` file mentioned earlier and the `build.gradle.kts` file that sits in the root. Both files are automatically generated when you create a new IntelliJ project and choose "IDE Plugin" as project type.

### Change kotlin to java

By default, the folder contains a src/main/kotlin folder. Either change 'kotlin' to 'java' or add a src/main/java folder. IntelliJ is okay with both.

#### plugin.xml

This is a plugin file that worked for me. I made annotations in it for non-default parts: 

```
<idea-plugin>
    <id>com.geertjankuip.carrot</id><!-- similar to group-->
    <name>Carrot</name><!-- similar to artifactId-->
    <vendor email="support@yourcompany.com" url="https://www.yourcompany.com">YourCompany</vendor>

    <description><![CDATA[
    Enter short description for your plugin here.<br>
    <em>most HTML tags may be used</em>
    ]]></description>

    <!-- this is provided by default-->
    <depends>com.intellij.modules.platform</depends> 
    
    <!-- without this one no Psi classes are loaded, imports will not work -->
    <depends>com.intellij.java</depends> 

    <!-- instead of the previous one you can use one of these as well, I don't know the difference -->
    <!--<depends>com.intellij.modules.lang</depends>-->
    <!--<depends>com.intellij.modules.java</depends>-->

    <actions>
        <action id="com.geertjankuip.plugin.HelloAction"
                class="com.geertjankuip.carrot.HelloAction"
                text="Say Hello"
                description="Greets the user.">            
            <add-to-group group-id="EditorPopupMenu" anchor="last"/>

            <!-- use this one to add the action to the Tools menu-->
            <!--<add-to-group group-id="ToolsMenu" anchor="last"/>-->
        </action>
    </actions>

    <!-- left this one as it was -- >
    <extensions defaultExtensionNs="com.intellij">

    </extensions>
</idea-plugin>
```

Summary: give it a good name, add additional dependencies to have all imports from the SDK work, and add the action section to integrate the plugin code in the interface.

#### build.gradle.kts

This file comes automatically and according to the documentation there is something with the difference between version 1.x and 2.x of "org.jetbrains.intellij". As far as problems arise, it will have to do with version numbers. Basically Gradle starts a sandbox IntelliJ version when you do 'Run Plugin' and this sandbox version is being downloaded in the version specified here. Changing the version results in a new download, which takes a lot of time. Anyway, I basically left the defaults as they were:

```
plugins {
    id("java")
    id("org.jetbrains.kotlin.jvm") version "1.9.25"
    id("org.jetbrains.intellij") version "1.17.4"
}

group = "com.geertjankuip"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

// Configure Gradle IntelliJ Plugin
// Read more: https://plugins.jetbrains.com/docs/intellij/tools-gradle-intellij-plugin.html
intellij {
    version.set("2024.1.7")
    type.set("IC") // Target IDE Platform

    // plugins.set(listOf(/* Plugin Dependencies */))
    plugins.set(listOf("java"))
}

tasks {
    // Set the JVM compatibility versions
    withType<JavaCompile> {
        sourceCompatibility = "17"
        targetCompatibility = "17"
    }
    withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
        kotlinOptions.jvmTarget = "17"
    }

    patchPluginXml {
        sinceBuild.set("241")
        untilBuild.set("243.*")
    }

    signPlugin {
        certificateChain.set(System.getenv("CERTIFICATE_CHAIN"))
        privateKey.set(System.getenv("PRIVATE_KEY"))
        password.set(System.getenv("PRIVATE_KEY_PASSWORD"))
    }

    publishPlugin {
        token.set(System.getenv("PUBLISH_TOKEN"))
    }
}
```

### Testing the plugin

To test what you have created, open the Gradle view at the right and press 'Run Plugin' under 'Run Configurations'. It might take a while the first time because Gradle needs to download a sandbox version of IntelliJ.

### Installing the plugin

when you are happy with the plugin you can package it with Gradle. I haven't one this yet but there are instructions.

