# Detecting assignments in the Abstract Syntax Tree

There are different ways of assigning values to variables. To be able to detect them some  basic knowledge of the AST is required, plus knowledge about traversal of the AST.

## TreePath

Without TreePath the AST can only be traversed top-down. If you have a specific tree object and you want to find its parent, you need TreePath. Every tree in the AST has an associated TreePath object that knows its parent TreePath, thus forming upward chains. Traversing upward is easier, because while a tree almost always has multiple children, it has only one parent.

In TreePathScanner, there is a field `path` that returns the TreePath object linked to the Tree. The method to retrieve it, when working in an extension of TreePathScanner is:

```
getCurrentPath(); => TreePath
```

To get the parent path, thus, moving upward towards the root, there is:

```
getCurrentPath().getParentPath(); => TreePath
```

To get the Tree linked to a TreePath, you have:

```
somepath.getLeaf(); => Tree
```

## Simple assignment detection

If you combine these things and want to climb up the Tree until you meet, for example, an AssignmentTree, you can do the following:

```
private boolean isInsideAssignment(TreePath path) {

    while (path!=null){

        if (path.getLeaf() instanceof JCTree.JCAssign) return true;
        if (path.getLeaf() instanceof JCTree.JCBlock) return false;

        path = path.getParentPath();
    }

    return false;
}
```

But this code only targets a specific type of assignment, and furthermore it doesn't tell you whether the Tree you are curious about is on the left side or the right side of the assigment. To start with the latter, we need to find the AssignmentTree (JCTree.JCAssign is the implementation) and then check its `lhs` and `rhs` fields.

Furthermore we need to do more then just detect whether or not our Tree is below some AssignmentTree, as we also need the TreePath object belonging to that AssignmentTree. The method below provides it. It returns null if no AssignmentTree (JCAssign) is found.

```
private TreePath findAssignment(TreePath path){

    while (path!=null){

        if (path.getLeaf() instanceof JCTree.JCAssign) break;
        if (path.getLeaf() instanceof JCTree.JCBlock) return null;

        path = path.getParentPath();
    }

    return path;
}
```

Once the TreePath of the AssignmentTree is available, it can be used to find out whether the MemberSelectTree items have been read or have been written to. While I first thought this would require going down in the structure (not using TreePath) I ended up using TreePath anyway. The reason is that going down requires consideration of what Tree type you encounter, with all sorts of partentheses, expressions etc. When going up you encounter just one type, namely TreePath itself.

```
private boolean isLeftAssignment(TreePath path, TreePath assignment){

    if (assignment.getLeaf().getKind()!= Tree.Kind.ASSIGNMENT) {
        throw new RuntimeException("Second parameter is not linked to an AssignmentTree.");}

    AssignmentTree assignmentTree = (AssignmentTree) assignment.getLeaf();

    while (path!=null && path.getParentPath()!=null){

        Tree up = path.getLeaf();
        Tree up2 = path.getParentPath().getLeaf();
        if (up2==assignmentTree){

            if (assignmentTree.getVariable() == up) {
                return true;
            } else return false;
        }
        path = path.getParentPath();
    }
    return false;
}
```

## Checking the Tree type

There are 3 ways to check the type of a tree. Let's say you have a variable which you know is of type Tree but you don't know what implementation it has:

- instanceof (`tree instanceof JCTree.JCVariableDecl vardec`)
- Tree.Kind (`tree.getKind() == Tree.Kind.VARIABLE`)
- JCTree.Tag (`tree.getTag() == JCTree.Tag.VARDEF`)

The first one can always be used, it asks for the specific implementation of the tree variable. The second one can be used for any compile type, Tree, JCTree or any of their subclasses. The third one can only be used if tree is of compile type JCTree, or any of its child classes. It does not work if compile type is Tree or any of its child interfaces.

A downside of the first one is that you need to involve a specific subclass of JCTree. Furthermore, while there are tons of JCTree subclasses that allow for refined checks, there are even more Tag values for even more refinement. For example, the subclass JCTree.JCUnary has 8 related Tag values (for `+,-,!,~,++_,--_,_++,_--)`. 

But if you want to access specific methods of the implemented type, this one is rather appropriate, because by using a new identifier (`vardec`) you have effectively changed the compile time type to match the runtime type.

A downside of the second is that you work in the public api of javac, using the official interfaces. Doing so prevents trouble (think of --add-export) but prevents the use of all sorts of useful methods that are only part of the hidden implementation. Once you use the 'forbidden' part of javac, there is no reason to stick to the public part of it, and the methods find in the subclasses of JCTree might be very effective.

A downside of the third option is that you can only use it if your compile type is JCTree. This might mean you have to cast type Tree to JCTree (`JCTree tree = (JCTree) treeinterface`). The benefit is that it is easy to group specific types of JCTree subclasses. ChatGPT gave this example:

```
switch (tree.getTag()) {
    case ASSIGN:
        ...
        break;
    case APPLY:
        ...
        break;
    case SELECT:
        ...
        break;
}
```

JCTrees that have ASSIGN as tag are of a different subclass than those having APPLY as tag, but you can now make them part of the same process. With `instanceof` you are limited to a single subclass. 

All in all, you might use different ways to check the type of a Tree, and as I am deep into forbidden javac territory I will mainly use `instancof` and `JCTree.Tag`.

## More advanced assignment detection

Assignments, or more specifically the writing to variables, can have multiple forms. In Java, can can group them under the following categories:

|Tree|JCTree|Tree.Kind|JCTree.Tag|
|---|---|---|---|
|AssignmentTree|JCAssign|ASSIGNMENT|ASSIGN|
|CompoundAssignmentTree|JCAssignOp|multiple|multiple|
|UnaryTree|JCUnary|multiple|multiple|

The Tree.Kind and JCTree.Tag values belonging to CompoundAssignmentTree/JCAssignOp are:

|Operator|Tree.Kind|JCAssignOP|
|---|---|---|
|*=|MULTIPLY_ASSIGNMENT|MUL_ASG|MUL_ASG|
|/=|DIVIDE_ASSIGNMENT|DIV_ASG|
|%=|REMAINDER_ASSIGNMENT|MOD_ASG|
|+=|PLUS_ASSIGNMENT|PLUS_ASG|
|-=|MINUS_ASSIGNMENT|MINUS_ASG|
|<<=|LEFT_SHIFT_ASSIGNMENT|SL_ASG|
|>>=|RIGHT_SHIFT_ASSIGNMENT|SR_ASG|
|>>>=|UNSIGNED_RIGHT_SHIFT_ASSIGNMENT|USR_ASG|
|&=|AND_ASSIGNMENT|BITAND_ASG|
|^=|XOR_ASSIGNMENT|BITXOR_ASG|
|&#124;=|XOR_ASSIGNMENT|BITOR_ASG|


