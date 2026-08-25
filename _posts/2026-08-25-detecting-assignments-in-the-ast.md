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




