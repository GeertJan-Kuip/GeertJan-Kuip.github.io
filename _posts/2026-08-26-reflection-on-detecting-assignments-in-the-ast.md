# Reflection on 'Detecting assignments in the AST'

I asked ChatGPT to review my blog post ['Detecting assignments in the AST'](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2026-08-25-detecting-assignments-in-the-ast.md). I find such reviews very helpful and here I will summarize what I got from it.

## Writing vs Mutating

For ChatGPT this distinction is fundamental, and generally I need to be more clear in my terminology. To differentiate between writing and mutation, Chat gives the following examples:

```
person.name = "Bob";
array[i] = 42;
list.add(x);
```

In the first example, `person` is being mutated while `name` is being written. In the second example, `array` is mutated and `array[i]` is written. In the third example, `list` is mutated, while there is not a variable that is written. It is correct to say that the mutated element is only read.

Chat looks at my code for method `isLeftAssignment` and remarks, correctly, that not everything on the left is assigned a new value ('write'). If `person.name` is on the left hand side of an assignment tree, only name is being written to.

## My own thought process on 'write' vs 'mutate'

One of the methods I created but not included in the blog post was named `isLast`. I created it to find the 'last' MemberSelectTree in a 'chain'. Let's say you have the following line:

```
this.testClass2.testClass3.testClass4.value = 4;
```

It is clear that only 'value' is being written to, and all the other elements in the chain are being mutated. So basically my intuïtion was to detect the last element of chains. Fun fact: in the AST this last element is actually the parent of all the previous elements.

This is what my `isLast` method, meant to have JCFieldAccess type as 'linked' argument, looks like:

```
private boolean isLast(TreePath path){

    TreePath parent = path.getParentPath();
    JCTree tree = (JCTree) parent.getLeaf();

    if (tree.getTag()== JCTree.Tag.SELECT) return false;
    if (tree.getTag()== JCTree.Tag.APPLY) {

        parent = parent.getParentPath();
        tree = (JCTree) parent.getLeaf();
    }

    while (tree.getTag()== JCTree.Tag.PARENS){
        parent = parent.getParentPath();
        tree = (JCTree) parent.getLeaf();
    }

    if (tree.getTag()== JCTree.Tag.SELECT) return false;

    return true;
}
```

So yes, I was aware of the difference between write and mutate, but I did not use this vocabulary in my text.

## Three questions

Chat proposes to distinguish three questions, namely the following:

- Is this expression part of an assignment's LHS?
- Is this expression the variable being written?
- Is some state being changed because of this expression?

The latter question has the most consequences, especially because any call to a method, domestic or foreign, can have any number of effects on 'state'. There doesn't need to be any of the JCAssign, JCAssignOp or JCUnary trees involved if you have something basic like `foo.setBar(x);`.

## Grouping JCAssign, JCAssignOp and JCBinary

The last method in my blog post is [`isLeftAssignment`](https://github.com/GeertJan-Kuip/GeertJan-Kuip.github.io/blob/main/_posts/2026-08-25-detecting-assignments-in-the-ast.md#better-version-for-checking-if-the-expression-is-on-the-left-or-right). Chat comments that the title of the method is misleading, as the method also handles JCBinary which doesn't write or assign anything.

The alternative proposal is to have two methods, one to check if some expression is on the left hand side (`isLeftOperand`) and the other being `isAssignmentTarget`. This makes sense, and actually `isLeftOperand` is a better method name than `isLeftAssignment`.

## What the left hand side can be

Chat refers to the JLS, chapter [15.26](https://download.java.net/java/early_access/jdk28/docs/specs/jls/jls-15.html#jls-15.26) to claim that any LHS in an assignment is one of the following three types:

- expression name
- field access
- array access

The JLS text says about the LHS operator: 

_"This operand may be a named variable, such as a local variable or a field of the current object or class, or it may be a computed variable, as can result from a field access (§15.11) or an array access (§15.10.3)."_

### Array

What I didn't discuss was the third case, where the lhs is something like `myArray[6]` or `someArray[i][j]`. The Tree type is ArrayAccessTree, the implementation type is JCArrayAccess, the Tree.Kind value is ARRAY_ACCESS and the JCTree.Tag is INDEXED. Two methods are important, namely:

- getExpression() -> returns Expression, everything but the index
- getIndex() -> returns Expression, which evaluates to some whole number

### Examples

```
b = 5; // expression name
this.b = 5; // field access
b[1] = 5; // array access
```

## Unary operators and compound assignment operators read and write

Chat stresses the fact that Unary operators, the four that are part of the list of 12 assignment operators (_++, _--, ++_, --_), are special in that they read and write at the same time. The same is true for compound assignment operators (+= etc).

## Method proposal

Chat proposes the following method, that specifically checks whether a specific Tree is an assignment target (being written to) or just one of the parent expressions that is being mutated but not written to:

```
private boolean isAssignmentTarget(TreePath origin, TreePath assignment) {
    Tree assignmentTree = assignment.getLeaf();

    TreePath path = origin;

    while (path != null && path.getParentPath() != null) {
        Tree child = path.getLeaf();
        Tree parent = path.getParentPath().getLeaf();

        if (parent == assignmentTree) {
            if (assignmentTree instanceof AssignmentTree a)
                return a.getVariable() == child;

            if (assignmentTree instanceof CompoundAssignmentTree a)
                return a.getVariable() == child;

            if (assignmentTree instanceof UnaryTree u)
                return u.getExpression() == child;
        }

        path = path.getParentPath();
    }

    return false;
}
```

Chat remarks that in case of Unary operators you need to look at parentheses, so the method needs some improvement, but the idea is clear. Let's say you have the following:

```
this.that.value = 5;
```

If you move up starting from Tree `this.that.value` you immediately hit its parent, `this.that.value = 5` which is an AssigmentTree. Using `getVariable()` gets you `this.that.value`, which tell you that this is the variable being written to. `this.that` and `this` are only mutated.

Btw this is also valid:

```
(this.that.value) = 5;
```

So you need to deal with parentheses, the method `com.sun.tools.javac.tree.skipParens` from TreeInfo might be useful here.

## What about initialization

The case not being discussed in my blog is the VariableTree with initialization, like:

```
String header = "This is the header";
```

Assignment takes place without this being one of my discussed tree types (AssignmentTree, CompoundAssignmentTree and UnaryTree). I need to think about what to do with it. The reason I overlooked it is because I am mainly interested in write operations that affect other classes than the class in which the write takes place itself.

## What about methods

Methods can affect state in the most unpredictable ways. Chat calls it a rabbit hole, trying to analyze what happens to 'state' during chains of method calls. I agree and do not know yet what to do with it.

One of the things I did was the following: if a method is part of a chain of member select trees, I try to get the return value of the method. Let's say the following expression is being used:

```
this.testClass2.testClass3.getTestClass4().value
```

To understand what happens you need to know the return value of method `getTestClass4()`. Generally, if a method is part of a member select chain but is not the last element in it, you know that it is about its return type. This type can be found, which makes it possible to know where the 'value' variable lives.

Of course this method `getTestClass4()` can do all sort of state-mutating things that cannot be easily detected.

## My thoughts

I am happy that my understanding of assignment, write and mutate developed during the process. The underlying reason to work on this is that I have a special interest not in 'state-change' per se, but in state change that is induced in class A but takes effect in state B, C or D or in all of them. This sort of hard-to-trace state change makes it hard to quickly understand how the 'wiring' of a program is done.

What I want is not only to see what the dependencies of a specific class are, but also via which paths these dependencies connect to the class I am studying. In the line `this.testClass2.testClass3.getTestClass4().value` I see that the Type/Symbol of identifier testClass3 is a dependency, **via** the Type/Symbol of testClass2. Just saying that TestClass3 is a dependency is not enough.

For Rich Hickey, Java developer turned Clojure inventor, the inherent complexity of OOP was reason to work on a functional language with little or no state at all. For me it is reason to work on static analyzers that guide you through the complex relations between classes in such a clear way that understanding the complex patterns becomes much easier. 





