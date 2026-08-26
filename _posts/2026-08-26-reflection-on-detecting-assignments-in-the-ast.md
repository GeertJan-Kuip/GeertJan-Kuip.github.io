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

## 3 questions

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

