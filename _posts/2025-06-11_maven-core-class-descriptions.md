# Maven Core class descriptions

I want to understand how Maven works so I'm gonna look at all classes. There are 368, or actually more, as some are generated during the generate-source phase by the Modello plugin (Model, Profile and Prerequisites from the org.apache.maven.api.model package). Total of packages is 52 I think.

It is gonna be extremely laborious work but I might learn a lot. My most important motivation to do this is to get a better understanding of complex codebases which will help me to build and/or find the right tools for quicker and more effective ways of code exploration.


## package org.apache.maven;

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

<details>
<summary><b>BuildAbort</b> : public class extends Error</summary>
<br/><i>A special throwable used to signal a graceful abort of the build.</i><br/><br/>
</details>

<details>
<summary><b>BuildFailureException</b> : public class extends Exception</summary>
<br/><i>One or more builds failed.</i><br/><br/>
</details>

<details>
<summary><b>DefaultMaven</b> : public class implements Maven</summary>
<br/>
Big class. Key terms: profiles, Mavensession, MavenExecutionResult, dependencyGraph, callListeners, validateLocalRepository, getExtensionComponents, getProjectScopedExtensionComponents, validatePrerequisitesForNonMavenPluginProjects, getAllProfiles.<br/><br/>

Interface Maven is an interface in the same folder. <br/><br/>
Three imports do not work: Model, Prerequisites and Profile. These are created during generate-sources phase by the modello plugin, based on .mdo file.<br/>


<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/DefaultMaven.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>DuplicateProjectException</b> : public class extends MavenExecutionException</summary>
<br/>
Small Exception class. Has method that gets and returns pom files of colliding projects.

<br/><i>Signals a collision of two or more projects with the same g:a:v during a reactor build.</i><br/><br/>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/MavenExecutionException.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>InternalErrorException</b> : public class extends MavenExecutionException</summary>
<br/><i>Signals an internal error in Maven itself, e.g. a programming bug.</i><br/><br/>
</details>

<details>
<summary><b>Maven</b> : public interface</summary>

<br/>
One method inside: MavenExecutionResult <b>execute</b>(MavenExecutionRequest request);

<br/><i>The main Maven execution entry point, which will execute a full Maven execution session. Implemented by DefaultMaven.</i><br/>
<br/><i>@see org.apache.maven.execution.MavenSession</i><br/><br/>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/Maven.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>MavenExecutionException</b> : public class extends Exception</summary>
<br/>
Has this method: public File <b>getPomFile</b>() {return pomFile;}<br/>
The Exception is created with pom file as argument.<br/><br/>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/MavenExecutionException.java">GitHub</a></i><br/><br/>
</details>

<details>
<summary><b>MissingProfilesException</b> : public class extends Exception</summary>
<br/><i>Signals that the user referenced one or more Maven profiles that could not be located in either the project or the settings.</i><br/><br/>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/MissingProfilesException.java">GitHub</a></i><br/><br/>
</details>

<details>
<summary><b>ProjectBuildFailureException</b> : public class extends BuildFailureException</summary>
<br/>
Has method public String <b>>getProjectId</b>() { return projectId; }<br/>
<br/><i>Exception which occurs when a normal (i.e. non-aggregator) mojo fails to execute. In this case, the mojo failed while executing against a particular project instance, so we can wrap the {@link MojoFailureException} with context information including projectId that caused the failure.</i><br/><br/>
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/ProjectBuildFailureException.java">GitHub</a></i><br/><br/>
</details>

<details>
<summary><b>ProjectCycleException</b> : public class extends BuildFailureException</summary>
<br/>
No Javadoc comments, no get method.<br/><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/ProjectCycleException.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>ReactorReader</b> : class ReactorReader implements MavenWorkspaceReader</summary>
<br/>
MavenWorkspaceReader comes from maven-imp module.<br/>
MavenWorkspaceReader is interface with one method:<br/>
-> Model <b>findModel</b>(Artifact artifact);<br/>
Note the 'Model' return value and the 'Artifact' argument.<br/>
MavenWorkspaceReader extends WorkspaceReader from org.eclipse.aether.repository
<br/><br/>
Public methods:<br/>
public WorkspaceRepository <b>getRepository</b>()<br/>
public File <b>findArtifact</b>(Artifact artifact)<br/>
public List<String> <b>findVersions</b>(Artifact artifact)<br/>
public Model <b>findModel</b>(Artifact artifact)<br/>

<br/><i>An implementation of a workspace reader that knows how to search the Maven reactor for artifacts, either as packaged jar if it has been built, or only compile output directory if packaging hasn't happened yet.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/ProjectCycleException.java">GitHub</a></i><br/>
</details>

 
<details>
<summary><b>RepositoryUtils</b> : public class</summary>
<br/>
Utility class with lots of imports from:<br/>
- org.apache.maven.artifact<br/>
- org.eclipse.aether.artifact<br/>
- org.eclipse.aether.graph<br/>
- org.eclipse.aether.repository<br/><br/>

Object types: ArtifactHandler, DefaultArtifactHandler, ArtifactHandlerManager, ArtifactRepository, ArtifactRepositoryPolicy, MavenArtifactProperties, DefaultRepositorySystemSession, RepositorySystem, RepositorySystemSession, Artifact, ArtifactProperties, ArtifactType, ArtifactTypeRegistry, DefaultArtifact, DefaultArtifactType, Dependency, DependencyFilter, DependencyNode, Exclusion, Authentication, LocalRepository, LocalRepositoryManager, Proxy, RemoteRepository, RepositoryPolicy, WorkspaceReader, WorkspaceRepository, AuthenticationBuilder.<br/>
<br/>Remarkable: the Maven Artifact type and the Eclipse Artifact type seemto differ, given this method:<br/>
-> public static org.apache.maven.artifact.Artifact <b>toArtifact</b>(Artifact artifact)<br/>
<br/><i><strong>Warning:</strong> This is an internal utility class that is only public for technical reasons, it is not part of the public API. In particular, this class can be changed or deleted without prior notice.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/RepositoryUtils.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>SessionScoped</b> : public @interface</summary>
<br/>
Marker annotation, @Retention(RUNTIME)

<br/><i>Indicates that annotated component should be instantiated before session execution starts and discarded after session execution completes.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/SessionScoped.java">GitHub</a></i><br/>
</details>


## package org.apache.maven.artifact;


<details>
<summary><b>DependencyResolutionRequiredException</b> : public class extends Exception</summary>
<br/>
Exception constructor has Artifact as argument.<br/><br/>

Message: "Attempted to access the artifact " + artifact + "; which has not yet been resolved"<br/>

<br/><i>Exception that occurs when an artifact file is used, but has not been resolved.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/DependencyResolutionRequiredException.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>InvalidRepositoryException</b> : public class extends Exception</summary>
<br/>
String repositoryId is an argument for the constructor.<br/>

<br/><i>Error constructing an artifact repository.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/InvalidRepositoryException.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.factory;

<details>
<summary><b>ArtifactFactory</b> : public interface (deprecated)</summary>
<br/>
Interface defining all sorts of constructors for Artifact object. DEPRECATED.<br/>
Selection of constructors:<br/>
-> Artifact createArtifact(String groupId, String artifactId, String version, String scope, String type);<br/>
-> Artifact createArtifactWithClassifier(String groupId, String artifactId, String version, String type, String classifier);<br/>
-> Artifact createDependencyArtifact(String groupId, String artifactId, VersionRange versionRange, String type, String classifier, String scope, String inheritedScope, boolean optional);<br/>
-> Artifact createBuildArtifact(String groupId, String artifactId, String version, String packaging);<br/>
-> Artifact createProjectArtifact(String groupId, String artifactId, String version);<br/>
-> Artifact createParentArtifact(String groupId, String artifactId, String version);<br/>
-> Artifact createPluginArtifact(String groupId, String artifactId, VersionRange versionRange);<br/>
-> Artifact createProjectArtifact(String groupId, String artifactId, String version, String scope);<br/>
-> Artifact createExtensionArtifact(String groupId, String artifactId, VersionRange versionRange);<br/>

<br/><i>ArtifactFactory - deprecated.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/ArtifactFactory.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>DefaultArtifactFactory</b> : public class implements ArtifactFactory</summary>
<br/>
Implementation of interface. The constructor with the complete set of arguments does all the work:<br/>
-> private Artifact createArtifact( String groupId, String artifactId, VersionRange versionRange, String type, String classifier, String scope, String inheritedScope, boolean optional);<br/><br/>
It is not a static factory, everything instance based. The constructor has an interesting argument:<br/>
-> public DefaultArtifactFactory(ArtifactHandlerManager artifactHandlerManager) { this.artifactHandlerManager = artifactHandlerManager; }<br/>


<br/><i>DefaultArtifactFactory.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/DefaultArtifactFactory.java">GitHub</a></i><br/>
</details>
