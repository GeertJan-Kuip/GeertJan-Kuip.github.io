## DIRRT and DOFSS annotations

DIRRT and DOFSS are acronyms I created and they represent the 10 annotations I need to learn for 1Z0-819. Five of them are meta-annotations, to be applied to other annotations, and the other five are regular annotations that you apply to code.

### DIRRT

#### Documented

Marker annotation. Annotations annotated by this will have their annotations published in Javadoc files.

#### Inherited

Marker annotation. Subclasses of annotations marked by Inherited will inherit the annotations of their parent class.

#### Retention

Retention annotation has one value element and its type is RetentionPolicy, an enum. RetentionPolicy has three possible constants:

- SOURCE
- CLASS
- RUNTIME

With SOURCE, the compiler will ignore the annotation type. With CLASS, annotation is included in the .class file but ignored during runtime. With RUNTIME, the annotation is available at runtime for reflection. Apply like ```@Retention(RetentionPolicy.CLASS)```.

#### Repeatable

This annotation indicates that an annotation can be used multiple times for the same element. There is a caveat: it will not compile if you just apply @Repeatable like you would normally do. Repeatable needs a containing class and should be applied like this:

```
@Repeatable(Dangers.class)
public @interface Danger{}
```

This 'Dangers.class' refers to a containg annotation you need to create, which is by convention named with the plural of the class it contains:

```
public @interface Dangers{
  Danger[] value();
}
```

Somehow they created a construction with an array of annotations that makes multiple appliances of a single annotation type possible.

#### Target

This is probably an important one. Target limits the code elements on which you can apply the annotation. When applying, you need to supply an array of ElementType enum values. Enum ElementType's source code is in java.lang.annotation and it's values are TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, ANNOTATION_TYPE, PACKAGE, TYPE_PARAMETER, TYPE_USE, MODULE and RECORD.

If you do not use @Target, the annotation is applicable to all elements except for those that belong to TYPE_PARAMETER and TYPE_USE. Practically it means that you cannot apply to:

- generic parameters in class- or method definitions
- generic type arguments
- cast expressions - ```(@SomeAnnotation String) obj```
- lambda parameters - ```(@SomeAnnotation String s)->s.length()```
- object creation - ```new @SomeAnnotation HashMap<>();```

The reason for this default is that TYPE_USE and TYPE_PARAMETER were only introduced in Java 8. Before Java 8, annotations were only possible in declaration contaxts. To guarantee backward compatibility, TYPE_USE and TYPE_PARAMETER had to be excluded.

Note: A typical Target annotation looks like this:

```
@Target({ElementType.TYPE, ELementType.METHOD, ElementType.PARAMETER etcetera})
```

### DOFSS

#### Deprecated

You add @Deprecated to notify a user during compilation that the method involved has a better alternative and that this method might be dropped somewhere in the future. Two optional values: 

```
String since() default "";
boolean forRemoval() default false;
```

It is strongly recommended that the reason for deprecating a program element be explained in the documentation, using the @deprecated tag (note the lowercase, this is a Javadoc annotation, not a Java annotation). If applicable, the documentation should also suggest and link to a recommended replacement API.

#### Override

Marker annotation. On the test, code will be provided in which method overrides are done wrongly. When you override a method, think of the following:

- same signature
- same *or broader* access modifier
- a covariant return type (same type or subtype of parent return type)
- do not declare new or broader exceptions. Narrower is allowed.

#### FunctionalInterface

Compiler reports error if the interface annotated by this is not a functional interface. Note that:

- a functional interface can be the extension of another interface
- a functional interface can have two (or more) abstract methods but only if those extra methods match the signature of java.lang.Object (think of boolean equals(Object o).

#### SuppressWarnings

Has ```String[] value();``` in its body. Note the two shorthands that are possible when applying this annotation to a method. 

The two possible values are "deprecation" and "unchecked". @SuppressWarnings does what it says, namely suppressing the warning(s) you specify. "unchecked" is about the use of raw types, "deprecation" is about the use of deprecated methods.

There are more possible warnings but these two are in the test.

#### SafeVarargs

Marker annotation. Can only be applied to constructors and methods that cannot be overridden. Methods must be static, private or final. The annotation indicates that no unsafe operations happen in the method and it suppresses unchecked compiler warnings for the varargs parameter in a method. 

Apply this annotation only on methods that have a varargs parameter and are final, static or private.

### Endnote

There is more to say about creating and applying annotations, this was just about ten annotations part of the exam. 










