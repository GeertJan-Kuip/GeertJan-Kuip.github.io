# Maven Core class descriptions

I want to understand how Maven works so I'm gonna look at all classes. There are 368 in the maven-core module alone, or actually more, as some are generated during the generate-source phase by the Modello plugin (Model, Profile and Prerequisites from the org.apache.maven.api.model package). Total of packages is 52 I think.

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
@Singleton<br/>
Big class. Many fields. Key terms: profiles, Mavensession, MavenExecutionResult, dependencyGraph, callListeners, validateLocalRepository, getExtensionComponents, getProjectScopedExtensionComponents, validatePrerequisitesForNonMavenPluginProjects, getAllProfiles.<br/><br/>

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
-> Artifact <b>createArtifact</b>(String groupId, String artifactId, String version, String scope, String type);<br/>
-> Artifact <b>createArtifactWithClassifier</b>(String groupId, String artifactId, String version, String type, String classifier);<br/>
-> Artifact <b>createDependencyArtifact</b>(String groupId, String artifactId, VersionRange versionRange, String type, String classifier, String scope, String inheritedScope, boolean optional);<br/>
-> Artifact <b>createBuildArtifact</b>(String groupId, String artifactId, String version, String packaging);<br/>
-> Artifact <b>createProjectArtifact</b>(String groupId, String artifactId, String version);<br/>
-> Artifact <b>createParentArtifact</b>(String groupId, String artifactId, String version);<br/>
-> Artifact <b>createPluginArtifact</b>(String groupId, String artifactId, VersionRange versionRange);<br/>
-> Artifact <b>createProjectArtifact</b>(String groupId, String artifactId, String version, String scope);<br/>
-> Artifact <b>createExtensionArtifact</b>(String groupId, String artifactId, VersionRange versionRange);<br/>

<br/><i>ArtifactFactory - deprecated.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/factory/ArtifactFactory.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>DefaultArtifactFactory</b> : public class implements ArtifactFactory</summary>
<br/>
Implementation of interface. The constructor with the complete set of arguments does all the work:<br/>
-> private Artifact <b>createArtifact</b>( String groupId, String artifactId, VersionRange versionRange, String type, String classifier, String scope, String inheritedScope, boolean optional);<br/><br/>
It is not a static factory, everything instance based. The constructor has an interesting argument:<br/>
-> public <b>DefaultArtifactFactory</b>(ArtifactHandlerManager artifactHandlerManager) { this.artifactHandlerManager = artifactHandlerManager; }<br/>

<br/><i>DefaultArtifactFactory.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/factory/DefaultArtifactFactory.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.handler;

<details>
<summary><b>DefaultArtifactHandler</b> : public class implements ArtifactHandler</summary>
<br/>
Class that creates ArtifactHandler instances. Lot of getters and setters. The constructor with all params is the following:<br/><br/>

-> public <b>DefaultArtifactHandler</b>(final String type, final String extension, final String classifier, final String directory, final String packaging, final boolean includesDependencies, final String language, final boolean addedToClasspath);<br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/handler/DefaultArtifactHandler.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.handler.manager;

 
<details>
<summary><b>ArtifactHandlerManager</b> : public interface</summary>
<br/>
Interface, one final static field and one method (second method is deprecated):<br/><br/>

-> String ROLE = ArtifactHandlerManager.class.getName();<br/>

-> ArtifactHandler <b>getArtifactHandler</b>(String type);<br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/handler/manager/ArtifactHandlerManager.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>DefaultArtifactHandlerManager</b> : public class extends AbstractEventSpy implements ArtifactHandlerManager</summary>
<br/>
@Singleton<br/>
This is the constructor declaration. Note the TypeRegistry argument:<br/><br/>
public <b>DefaultArtifactHandlerManager</b>(TypeRegistry typeRegistry)<br/><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/handler/manager/DefaultArtifactHandlerManager.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>LegacyArtifactHandlerManager</b> : public class extends AbstractEventSpy</summary>
<br/>
@Singleton<br/>
This is the constructor:<br/><br/>
public LegacyArtifactHandlerManager(Map<String, ArtifactHandler> artifactHandlers) {this.artifactHandlers = requireNonNull(artifactHandlers);}

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/LegacyArtifactHandlerManager.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.repository;

<details>
<summary><b>DefaultRepositoryRequest</b> : public class implements RepositoryRequest</summary>
<br/>
Four instance fields:<br/><br/>
private boolean offline;<br/>
private boolean forceUpdate;<br/>
private ArtifactRepository localRepository;<br/>
private List&ltArtifactRepository&gt remoteRepositories;<br/><br/>

Constructor creates a shallow copy of the specified repository request.<br/> 

public DefaultRepositoryRequest(RepositoryRequest repositoryRequest) {<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;setLocalRepository(repositoryRequest.getLocalRepository());<br/>
&nbsp;&nbsp;&nbsp;&nbsp;setRemoteRepositories(repositoryRequest.getRemoteRepositories());<br/>
&nbsp;&nbsp;&nbsp;&nbsp;setOffline(repositoryRequest.isOffline());<br/>
&nbsp;&nbsp;&nbsp;&nbsp;setForceUpdate(repositoryRequest.isForceUpdate());<br/>
}<br/>

<i>Collects basic settings to access the repository system.</i><br/>

## package org.apache.maven.artifact.repository;
<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/DefaultRepositoryRequest.java">GitHub</a></i><br/>
</details>

 
<details>
<summary><b>MavenArtifactRepository</b> : public class implements ArtifactRepository</summary>
<br/>
These are the fields. Lots of getters and setters:<br/><br/>
private static final String LS = System.lineSeparator();<br/>
private String id;<br/>
private String url;<br/>
private String basedir;<br/>
private Path basedirPath;<br/>
private String protocol;<br/>
private ArtifactRepositoryLayout layout;<br/>
private ArtifactRepositoryPolicy snapshots;<br/>
private ArtifactRepositoryPolicy releases;<br/>
private Authentication authentication;<br/>
private Proxy proxy;<br/>
private List&ltArtifactRepository&gt mirroredRepositories = Collections.emptyList();<br/>
private boolean blocked;<br/><br/>

<i>Abstraction of an artifact repository. Artifact repositories can be remote, local, or even build reactor or IDE workspace.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/MavenArtifactRepository.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>RepositoryCache</b> : public interface (Deprecated)</summary>
<br/>
@Deprecated<br/><br/>

<i>Caches auxiliary data used during repository access like already processed metadata. The data in the cache is meant for exclusive consumption by the repository system and is opaque to the cache implementation.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/RepositoryCache.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>RepositoryRequest</b> : public interface</summary>
<br/>

<i>Collects basic settings to access the repository system.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/RepositoryRequest.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.repository.layout;

<details>
<summary><b>DefaultRepositoryLayout</b> : public class implements ArtifactRepositoryLayout</summary>
<br/>
@Singleton
<br/>
This class has methods in it that compose path-like strings using StringBuilder, with groupId, artifactId, baseVersion etc.
<br/><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/layout/DefaultRepositoryLayout.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.repository.metadata.io;

<details>
<summary><b>DefaultMetadataReader</b> : public class implements MetadataReader</summary>
<br/>
@Singleton
<br/>
Uses library org.apache.maven.artifact.repository.metadata.Metadata but MetaData class is not in it. Can be a modello thing. Has all sorts of read methods that read metadata in various types.<br/><br/>

<i>Handles deserialization of metadata from some kind of textual format like XML.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/metadata/io/DefaultMetadataReader.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>MetadataParseException</b> : public class extends IOException</summary>
<br/>
Error message has int lineNumber and int columnNumber as arguments. Has get methods for both.
<br/><br/>

<i>Signals a failure to parse the metadata due to invalid syntax (e.g. non well formed XML or unknown elements).</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/metadata/io/MetadataParseException.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>MetadataReader</b> : public interface</summary>
<br/>
All about reading metadata and whether to be strict with parsing.
<br/><br/>

<i>Handles deserialization of metadata from some kind of textual format like XML.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/repository/metadata/io/MetadataReader.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.artifact.resolver.filter;

<details>
<summary><b>AbstractScopeArtifactFilter</b> : abstract class implements ArtifactFilter</summary>
<br/>
Fields:<br/>
private boolean compileScope;<br/>
private boolean runtimeScope;<br/>
private boolean testScope;<br/>
private boolean providedScope;<br/>
private boolean systemScope;<br/>
<br/>

<i>Filter to only retain objects in the given artifactScope or better.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/AbstractScopeArtifactFilter.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>AndArtifactFilter</b> : public class implements ArtifactFilter</summary>
<br/>
One field:<br/>
private Set&ltArtifactFilter&gt filters;
<br/>

<i>Apply multiple filters.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/AndArtifactFilter.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>CumulativeScopeArtifactFilter</b> : public class extends AbstractScopeArtifactFilter</summary>
<br/>
One field:<br/>
private Set&ltString&gt scopes;
<br/>

<i>Filter to only retain objects in the given scope or better. This implementation allows the accumulation of multiple scopes and their associated implied scopes, so that the user can single step. This should be a more efficient implementation of multiple standard {@link ScopeArtifactFilter} instances ORed together.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/CumulativeScopeArtifactFilter.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>ExcludesArtifactFilter</b> : public class extends IncludesArtifactFilter</summary>
<br/>

<i>Filter to exclude from a list of artifact patterns.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/ExcludesArtifactFilter.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>ExclusionArtifactFilter</b> : public class implements ArtifactFilter</summary>
<br/>

<i>Filter to exclude from a list of artifact patterns.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/ExclusionArtifactFilter.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>ExclusionSetFilter</b> : public class implements ArtifactFilter</summary>
<br/>
One field:<br/>
private Set&ltString&gt excludes;<br/>

<i>Filter to exclude from a list of artifact patterns.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/ExclusionSetFilter.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>IncludesArtifactFilter</b> : public class implements ArtifactFilter</summary>
<br/>
One field:<br/>
private final Set&ltString&gt patterns;<br/>

<i>Filter to include from a list of artifact patterns.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/IncludesArtifactFilter.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>ScopeArtifactFilter</b> : public class extends AbstractScopeArtifactFilter</summary>
<br/>
One field:<br/>
private final String scope;<br/>

<i>Filter to only retain objects in the given artifactScope or better.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/artifact/resolver/filter/ScopeArtifactFilter.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.bridge;

<details>
<summary><b>MavenRepositorySystem</b> : public class</summary>
<br/>
@Singleton<br/>
Huge class, no Javadoc. I find it remarkable that it has (overloaded) methods that create all sorts of Artifact objects. Class DefaultArtifactFactory has all these methods as well.<br/>
Other remarkable thing: hardcoded value for repository:<br/>
-> public static final String DEFAULT_REMOTE_REPO_URL = "https://repo.maven.apache.org/maven2";<br/><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/bridge/MavenRepositorySystem.java">GitHub</a></i><br/>
</details>


## package org.apache.maven.classrealm;

<details>
<summary><b>ArtifactClassRealmConstituent</b> : class implements ClassRealmConstituent</summary>
<br/>
One field:<br/>
private final Artifact artifact;<br/><br/>

This class lets you inquire the specific artifact object it contains using get methods.<br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/ArtifactClassRealmConstituent.java">GitHub</a></i><br/>
</details>

 
<details>
<summary><b>ClassRealmConstituent</b> : public interface</summary>
<br/>
Interface describing ArtifactClassRealmConstituent.<br/><br/>
<i>Describes a constituent of a class realm.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/ClassRealmConstituent.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>ClassRealmManager</b> : public interface</summary>
<br/>

<i>Manages the class realms used by Maven. <strong>Warning:</strong> This is an internal utility interface that is only public for technical reasons, it is not part of the public API. In particular, this interface can be changed or deleted without prior notice.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/ClassRealmManager.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>ClassRealmManagerDelegate</b> : public interface</summary>
<br/>

<i>ClassRealmManagerDelegate is used to perform addition configuration of class realms created by ClassRealmManager.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/ClassRealmManagerDelegate.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>ClassRealmRequest</b> : public interface</summary>
<br/>
Contains an enum 'RealmType', values Core, Project, Plugin, Extension.<br/><br/>
<i>Describes the requirements for a new class realm.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/ClassRealmRequest.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>DefaultClassRealmManager</b> : public class implements ClassRealmManager</summary>
<br/>
@Singleton<br/>
Has methods to create ClassRealms. It is a big class.
<br/><br/>
<i>Manages the class realms used by Maven. <strong>Warning:</strong> This is an internal utility class that is only public for technical reasons, it is not part of the public API. In particular, this class can be changed or deleted without prior notice.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/DefaultClassRealmManager.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>DefaultClassRealmRequest</b> : class implements ClassRealmRequest</summary>
<br/>
POJO, getters and setters. This is the constructor:<br/>
->     DefaultClassRealmRequest( RealmType type, ClassLoader parent, List<String> parentImports, Map<String, ClassLoader> foreignImports, List<ClassRealmConstituent> constituents)<br/><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/classrealm/DefaultClassRealmRequest.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.configuration;

<details>
<summary><b>BasedirBeanConfigurationPathTranslator</b> : public class implements BeanConfigurationPathTranslator</summary>
<br/>
Just one import:<br/>
-> import java.io.File;<br/>
And one method:<br/>
public File translatePath(File path)<br/>
<br/>
<i>A path translator that resolves relative paths against a specific base directory.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BasedirBeanConfigurationPathTranslator.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>BeanConfigurationException</b> : public class extends Exception</summary>
<br/>

<i>Thrown when a bean couldn't be configured.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BeanConfigurationException.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>BeanConfigurationPathTranslator</b> : public interface</summary>
<br/>
Implemented by BasedirBeanConfigurationPathTranslator.<br/><br/>

<i>Postprocesses filesystem paths. For instance, a path translator might want to resolve relative paths given in the bean configuration against some base directory.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BeanConfigurationPathTranslator.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>BeanConfigurationRequest</b> : public interface</summary>
<br/>

<i>A request to configure a bean from some configuration in the POM or similar.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BeanConfigurationRequest.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>BeanConfigurationValuePreprocessor</b> : public interface</summary>
<br/>

<i>Preprocesses a value from a bean configuration before the bean configurator unmarshals it into a bean property. A common use case for such preprocessing is the evaluation of variables within the configuration value.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BeanConfigurationValuePreprocessor.java">GitHub</a></i><br/>
</details>

<details>
<summary><b>BeanConfigurator</b> : public interface</summary>
<br/>

<i>Unmarshals some textual configuration from the POM or similar into the properties of a bean. This component works similar to the way Maven configures plugins from the POM, i.e. some configuration like {@code <param>value</param>} is mapped to an equally named property of the bean and converted. The properties of the bean are supposed to either have a public setter or be backed by an equally named field (of any visibility).</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/BeanConfigurator.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>DefaultBeanConfigurationRequest</b> : public class implements BeanConfigurationRequest</summary>
<br/>
This is an interesting class, seems to deal with finding and configuring plugins.<br/><br/>
Constructor:<br/>
public DefaultBeanConfigurationRequest setConfiguration(<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;Model model, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;String pluginGroupId, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;String pluginArtifactId, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;String pluginExecutionId<br/>
) <br/><br/>

<i>Javadoc constructor: Sets the configuration to the configuration taken from the specified build plugin in the POM. First, the build plugins will be searched for the specified plugin, if that fails, the plugin management section will be searched.</i><br/>

<i>Javadoc class: Basic bean configuration request.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/DefaultBeanConfigurationRequest.java">GitHub</a></i><br/>
</details>

## package org.apache.maven.configuration.internal;

<details>
<summary><b>DefaultBeanConfigurator</b> : public class implements BeanConfigurator</summary>
<br/>
@Singleton<br/>
Has nested static classes in it, 'XmlConverter' and 'PathConverter'.<br/><br/>

<i><strong>Warning:</strong> This is an internal class that is only public for technical reasons, it is not part of the public API. In particular, this class can be changed or deleted without prior notice.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/internal/DefaultBeanConfigurator.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>EnhancedConverterLookup</b> : class implements ConverterLookup</summary>
<br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/internal/EnhancedConverterLookup.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>EnhancedComponentConfigurator</b> : public class extends BasicComponentConfigurator</summary>
<br/>
@Singleton<br/><br/>
<i>A component configurator which can leverage the {@link EnhancedConfigurationConverter} and {@link EnhancedConverterLookup}.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/internal/EnhancedComponentConfigurator.java">GitHub</a></i><br/>
</details>


<details>
<summary><b>EnhancedConfigurationConverter</b> : class extends ObjectWithFieldsConverter</summary>
<br/>

<i>An enhanced {@link ObjectWithFieldsConverter} leveraging the {@link TypeAwareExpressionEvaluator} interface.</i><br/>

<i><a href="https://github.com/apache/maven/blob/master/impl/maven-core/src/main/java/org/apache/maven/configuration/internal/EnhancedConfigurationConverter.java">GitHub</a></i><br/>
</details>

