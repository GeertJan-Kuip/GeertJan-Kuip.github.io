# Detecting assignments in the Abstract Syntax Tree

There are different ways of assigning values to variables. To be able to detect them some  basic knowledge of the AST is required, plus knowledge about traversal of the AST.

## TreePath

Without TreePath the AST can only be traversed top-down. If you have a specific tree object and you want to see of wh ich tree it is a child, you need TreePath. Every tree in the AST has an associated TreePath object that knows its parent TreePath, thus forming upward chains.

In TreePathScanner, there is a field `path` that returns the TreePath object linked to the Tree. The method to retrieve it, when working in an extension of TreePathScanner is:

```
getCurrentPath(); => TreePath
```

To get the parent path, thus, moving upward towards the root, there is:

```
getParentPath(); => TreePath
```

To get the Tree linked to a TreePath, you have:

```
getLeaf(); => Tree
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


