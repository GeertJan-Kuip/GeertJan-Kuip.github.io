# Maven Core class descriptions

I want to understand how Maven works so I'm gonna look at all classes.


### package org.apache.maven;

#### public abstract class **AbstractMavenLifecycleParticipant**

_Allows core extensions to participate in Maven build session lifecycle.

All callback methods (will) follow beforeXXX/afterXXX naming pattern to indicate at what lifecycle point it is being called.

@see <a href="https://maven.apache.org/examples/maven-3-lifecycle-extensions.html">example</a>
@see <a href="https://issues.apache.org/jira/browse/MNG-4224">MNG-4224</a>
@since 3.0-alpha-3_
