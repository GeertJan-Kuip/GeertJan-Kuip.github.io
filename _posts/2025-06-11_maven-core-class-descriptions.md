# Maven Core class descriptions

I want to understand how Maven works so I'm gonna look at all classes. There are 368, or actually more, as some are generated during the generate-source phase by the Modello plugin (Model, Profile and Prerequisites from the org.apache.maven.api.model package). Total of packages is 52 I think.

It is gonna be extremely laborious work but I might learn a lot. My most important motivation to do this is to get a better understanding of complex codebases which will help me to build and/or find the right tools for quicker and more effective ways of code exploration.


#### package org.apache.maven;

<details>
<summary><b>AbstractMavenLifecycleParticipant</b> : public abstract class</summary>

<br/>
The classes implementing this class reside in the test directory.  
<br/><br/>
<i>Allows core extensions to participate in Maven build session lifecycle.</i><br/>
<i>All callback methods (will) follow beforeXXX/afterXXX naming pattern to indicate at what lifecycle point it is being called.</i><br/><br/>
<i>@see <a href="https://maven.apache.org/examples/maven-3-lifecycle-extensions.html">example</a></i> 
<i>@see <a href="https://issues.apache.org/jira/browse/MNG-4224">MNG-4224</a></i> 
<i>@since 3.0-alpha-3</i>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/AbstractMavenLifecycleParticipant.java">GitHub</a></i><br/><br/>

</details>