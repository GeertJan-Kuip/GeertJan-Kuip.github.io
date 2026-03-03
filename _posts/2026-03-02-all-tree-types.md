# All Tree types

I want to understand in detail how the Java Abstract Syntax Tree is structured. Therefore I will have a close look at all the 77 Tree interfaces in com.sun.source.tree and try to understand how those different Tree objects typically form AST's.

## Tree

The mother of all Tree types has two methods, namely `getKind()` and `accept(..)`. The former returns a Kind object, Kind is an internal enum in Tree. It has,  if I count correctly, 107 values. This is more than the number of interfaces. This means there is a one to many mapping, once you know the Tree type you can use Kind for an extra level of specificity. Or you can directly ask for Kind and then inquire the Tree type belonging to it. Every enum has an associated value of type Class, this is actually the specific Tree interface. A few examples:

```
TYPE_PARAMETER(TypeParameterTree.class),
VARIABLE(VariableTree.class),
WHILE_LOOP(WhileLoopTree.class),
```

As expected, enum values can have the same interface type as associated value:

```
POSTFIX_INCREMENT(UnaryTree.class),
POSTFIX_DECREMENT(UnaryTree.class),
PREFIX_INCREMENT(UnaryTree.class),
PREFIX_DECREMENT(UnaryTree.class),
UNARY_PLUS(UnaryTree.class),
UNARY_MINUS(UnaryTree.class),
BITWISE_COMPLEMENT(UnaryTree.class),
LOGICAL_COMPLEMENT(UnaryTree.class),

UNBOUNDED_WILDCARD(WildcardTree.class),
EXTENDS_WILDCARD(WildcardTree.class),
SUPER_WILDCARD(WildcardTree.class),
```

If we look further in the internal enum, we see this:

```
        Kind(Class<? extends Tree> intf) {
            associatedInterface = intf;
        }

        public Class<? extends Tree> asInterface() {
            return associatedInterface;
        }

        private final Class<? extends Tree> associatedInterface;
```

The `associatedInterface` variable is of type `Class<? extends Tree>`, which means that if you know the Kind value of a Tree, you can inquire the type of Tree. The other way around logically is not possible because one Tree type can correspond to multiple Kind values. 

The way to get the Kind value of a specific Tree instance is done with the abstract 'getKind` method:

`` 
    Kind getKind();
```

This method is implemented in the internal JC objects. Remember, JCTree is the class implementing all the Tree interfaces, and it has a range of static subclasses representing the variety of Tree types. For example, in static inner class JCClassDecl we find this implementation:

```
        @DefinedBy(Api.COMPILER_TREE)
        public Kind getKind() {
            if ((mods.flags & Flags.ANNOTATION) != 0)
                return Kind.ANNOTATION_TYPE;
            else if ((mods.flags & Flags.INTERFACE) != 0)
                return Kind.INTERFACE;
            else if ((mods.flags & Flags.ENUM) != 0)
                return Kind.ENUM;
            else if ((mods.flags & Flags.RECORD) != 0)
                return Kind.RECORD;
            else
                return Kind.CLASS;
        }
```

Look how the Flags class (`com.sun.tools.javac.code.Flags`) and a flag value (primitive type 'long') are involved here. An expression like `((mods.flags & Flags.ANNOTATION) != 0)` basically checks if a specific bit in the 64 bit flag value indicating that it is a annotation interface (specific type of a class) is set to 1. Note that this use of the Flag class is done both in JCTree and in Symbol. In Symbol the flags_field instance field contains the 64-bit value, in JCTree the 64-bit field is found not as an instance field but as a field in a few subclasses, more specifically JCBlock and JCModifiers.

The main point is that there exist multiple categorizations side by side, each tailor-made for the underlying structure that needs to be described. 'Kind' is an enum specific for the AST, 'Flags' is a class containing final static int values with names written in capital letters. It is associated with the 64-bit value that is found both in JCTree  and with Symbol. 

Last thing: the Tree interface has an abstract accept method as well, implemented in the JC classes:

```
    <R,D> R accept(TreeVisitor<R,D> visitor, D data);
```

## The structure of the AST

The AST is a structured tree with specific rules that describe which tree type can be a child node of which tree type. This structure can be deduced from the abstract methods in the various Tree interfaces. As an example, here is DoWhileLoopTree:

```
public interface DoWhileLoopTree extends StatementTree {

    ExpressionTree getCondition();

    StatementTree getStatement();
}
```

Apparently every DoWhileLoopTree has a condition, represented by ExpressionTree, and a statement, represented by StatementTree. DoWhileLoopTree itself is an extension of StatemenTree. 

### ExpressionTree

ExpressionTree is a marker interface, meaning it has an empty body. It is the superinterface of many other of the 77 Tree interfaces. It is used as a return type, when you do not know what type of expression is actually in the code. These are the 22 subtypes:

```
AnnotatedTypeTree
AnnotationTree
ArrayAccesTree
AssignmentTree
BinaryTree
CompoundAssignmentTree
ConditionalExpressionTree
ErroneousTree
IdentifierTree
InstanceOfTree
LambdaExpressionTree
LiteralTree
MemberReferenceTree
MemberSelectTree
MethodInvocationTree
NewArrayTree
NewClassTree
ParenthesizedTree
StringTemplateTree
SwitchExpressionTree
TypeCastTree
UnaryTree
```

It is important to note that the 'Kind' value of a Tree is often required to understand what the code in this tree does. If, for example, we take BinaryTree, we have this code:

```
public interface BinaryTree extends ExpressionTree {

    ExpressionTree getLeftOperand();

    ExpressionTree getRightOperand();
}
```

To know what operator is being used, you need Kind. This is what the javadoc comment says as well: _A tree node for a binary expression. Use getKind to determine the kind of operator._

These Kind values exist in the Kind enum:

```
        MULTIPLY(BinaryTree.class),
        DIVIDE(BinaryTree.class),
        REMAINDER(BinaryTree.class),
        PLUS(BinaryTree.class),
        MINUS(BinaryTree.class),
        LEFT_SHIFT(BinaryTree.class),
        RIGHT_SHIFT(BinaryTree.class),
        UNSIGNED_RIGHT_SHIFT(BinaryTree.class),
        LESS_THAN(BinaryTree.class),
        GREATER_THAN(BinaryTree.class),
        LESS_THAN_EQUAL(BinaryTree.class),
        GREATER_THAN_EQUAL(BinaryTree.class),
        EQUAL_TO(BinaryTree.class),
        NOT_EQUAL_TO(BinaryTree.class),
        AND(BinaryTree.class),
        XOR(BinaryTree.class),
        OR(BinaryTree.class),
        CONDITIONAL_AND(BinaryTree.class),
        CONDITIONAL_OR(BinaryTree.class),
```

### Leafs

Some Tree types are so called 'leafs', meaning they do not have any more Tree objects beneath them. An example is LiteralTree:

```
public interface LiteralTree extends ExpressionTree {
    /**
     * Returns the value of the literal expression.
     * The value will be a boxed primitive value, a String, or {@code null}.
     * @return the value
     */
    Object getValue();
}
```

The Javaddoc comment says: _A tree node for a literal expression. Use getKind to determine the kind of literal._ These are the Kind values related to LiteralTree (all are primitives except String and null):

```
        INT_LITERAL(LiteralTree.class),
        LONG_LITERAL(LiteralTree.class),
        FLOAT_LITERAL(LiteralTree.class),
        DOUBLE_LITERAL(LiteralTree.class),
        BOOLEAN_LITERAL(LiteralTree.class),
        CHAR_LITERAL(LiteralTree.class),
        STRING_LITERAL(LiteralTree.class),
        NULL_LITERAL(LiteralTree.class),
```

### StatementTree

