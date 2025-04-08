## Annotations

Some facts about annotations. I left out some rules for applying annotations (allowed types, dealing with array types) and I din't explain @Repeateble. It is in a [previous post](https://geertjan-kuip.github.io/2025/03/08/dirrt-and-dofss-annotations.html).

- An annotation can be declared in a class body. The declaration is implicitly static and can be called by @classname.annotationnam().
- A marker annotation can contain constants but has no elements.
- A default value must be a non-null constant expression.
- Elements are implicitly public and abstract.
- Constants are implicitly public, static and final.
- When applying an annotation to an expression, a cast operation including the Java type is required.

To apply an annotation without its name three conditions must be met:
- There must be an element named value(), which may be optional or required.
- The annotation declaration must not contain any other elements that are required.
- The annotation usage must not provide values for any other element.

### @Target

There is overlap between ElementTypes, namely the following:

#### TYPE and TYPE_USE

TYPE_USE can be applied anywhere there is a Java type **declared** or **used**. TYPE is used for declarations of classes, enums, interfaces and annotations. If you have TYPE_USE, you can do without TYPE because of the overlap.

#### TYPE and ANNOTATION_TYPE

ANNOTATION_TYPE only includes annotation declarations, while TYPE includes classes, interfaces and enums as well.

_Generally, declarations of annotations, classes, enums and interfaces are covered by multiple ElementTypes.

#### More about TYPE_USE

- The most complex of all ElementTypes.
- Can be used anywhere there is a Java type.
- Can be used on methods but only if they return a value. A void method requires METHOD.
- When using it ahead of a literal or an expression, it must have the form of a cast: ```int remaining = (@Technical int) 10.0;``` or ```return (@Floats boolean) t.test(12);```

#### PARAMETER

Applies to the parameters of _constructors, methods and lambdas._

#### METHOD and CONSTRUCTOR

ElementType METHOD does not apply constructor declarations, therefore you need CONSTRUCTOR and vice versa.

#### FIELD

Field applies not only to instance and static variables but also to enum values.

#### PACKAGE, MODULE an TYPE_PARAMETER

These are not on the exam.

### About the other annotations

- For @Retention the default is RetentionPolicy.CLASS. Annotations not available at runtime by default.
- @Deprecated has two optional elements, namely ```String since() default ""; ``` and ```boolean forRemoved() default false; ```.
- @Deprecated can be applied to all sorts of declarations (methods, variables, classes etc.). Also to Annotations.
- When using @Deprecated, add Javadoc with @deprecated with reason and suggestion for alternatives.
- @SuppressWarnings has ```String[] value(); ``` as element. The two to know are "deprecation" and "unchecked", the first to suppress deprecated warnings, the second for warnings related to raw types. Think of ```new ArrayList() ```.
- @SafeVarargs can **only** be applied to methods that cannot be overridden (final, private or static).
- @SafeVarargs suppresses warnings related to varags method parameters. 
- @SafeVarargs gives compiler error when applied to method without varargs parameter or method that is overridable.

### For personal use

List of mistakes I made in review questions chapter 13:

- It is @Documented, not @Document
- ```int temperature;``` is not valid constant or element, must be ```int temperature()``` or ```int temperature=12```
- To use annotation without name (shorthand) its only required element must be named value().
- Check always if the name of an element is value(), relevant for shorthand notation.
- @SafeVarargs can be applied to both methods and constructors
- Providing documentation is (always) a reason to add annotations
- When using TYPE_USE on literals or expressions, use it as a cast with a type between parentheses like ```(@Annotation Integer) 10%3```
- Default of @Retention is RetentionPolicy.CLASS. This means annotations are no part of runtime.





