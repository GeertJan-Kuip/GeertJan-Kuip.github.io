# Specifications


NOTE: this text is under construction. Please ignore.

Different levels

- Application
Modular or not
Type of modularity
UML graph modules or packages
Independent modules
Where main method/ entry point (check pom, META)
Dependencies
How it should be packaged
Tests available

- Module (can be unnamed module)
Has entry point
Package uml
Is independent
Dependent on/ requires
Independent packages
Most central package
Abstract dependency inversions
Circular dependencies

- Any two packages
Relationships
Points of contact
generalization of points of contact
Abstract bridges


- Package
Class uml
entry
Exposed classes
Who depends on it
On who does it depend
Meaning of interfaces/abstract classes
Exported, required

- Class
Centrality
Aliases
Multiple or single
Singleton
Injected/used in its abstract form
Methods being externally called
Type of class (domain, value, utility, composition root, controller, datatransformer)
Dependencies, two way
Patterns found related to this class

- Field
How injected
Abstract or not
Where instantiated
Role in class (dataprovider, adding functionality)
List of memberselects, typical structure of memberselect
Dependencies of the field class
Other classes containing this instance
Other classes having this type of dependency

- All fields
How injected
Which ones are abstract
Which have custom types?
Which are from external libraries?
Instantiated on same site?
All from same package, module

- Method
Purely functional?
Which state is affected
New dependencies created
Returns abstract form of some implementation

- All methods
Indicates utility class? (purely functional)
Typical return types
Typical argument types
Percentage static methods
Externally accessible methods
Methods accessed from outside (other package, other module)





