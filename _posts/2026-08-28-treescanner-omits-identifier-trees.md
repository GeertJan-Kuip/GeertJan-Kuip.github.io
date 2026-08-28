# TreeScanner omits methods returning Name object 

I just noticed that when you walk the AST using an extension of TreeScanner (or TreePathScanner, doesn't matter here), that some objects in the structure will not be scanned. In both VariableTree and MemberSelectTree, you would expect certain IdentifierTree objects to be part of the scanning process but they are not included in the scan, or actually they do not even exist. I'll explain.

## VariableTree

VariableTree is a tree for statements like the following:

```
Integer a = 6;
String str = "Cook";
Car car = new Car();
```

The five methods in the VariableTree interface are the following:

```
public interface VariableTree extends StatementTree {

	ModifiersTree getModifiers();

	Name getName();

	ExpressionTree getNameExpression();

	Tree getType();

	ExpressionTree getInitializer();
}
```

If class TreeScanner you can see how the visitor scans VariableTree objects. The relevant code here is:

```
    @Override
    public R visitVariable(VariableTree node, P p) {
        R r = scan(node.getModifiers(), p);
        r = scanAndReduce(node.getType(), p, r);
        r = scanAndReduce(node.getNameExpression(), p, r);
        r = scanAndReduce(node.getInitializer(), p, r);
        return r;
    }
```

As you see, only four of the five methods are being executed. `Name getName();` is omitted. My intuition would be that this getName() method should return an IdentifierTree (it doesn't, it returns a Name object) and that when I would override the visitIdentifierTree method in my extended scanner, it would pick up the name of the declared variable. But it doesn't.

Let's have a simple example, a variable declaration within a method body:

```
Integer myInteger = 2;
```

When I print out every tree that the scanner visits, this is the output (I added the Tree type between parentheses):

```
Integer myInteger = 2 (VariableTree)
(ModifiersTree)
Integer (IdentifierTree)
2 (LiteralTree)
```

Note that because there are no modifiers applied, the ModifiersTree return value prints nothing. But note also that `myInteger` is not found in the output, even though two of the methods of VariableTree, namely `Name getName()` and `ExpressionTree getNameExpression()` suggest they could return `myInteger`.

If I do not rely on the visitVariable method but instead apply all VariableTree's five methods myself, I get the floowing output (method name followed by return value):

```
  getModifiers() => 
  getName() => myInteger
  getNameExpression() => null
  getType() => Integer
  getInitializer() => 2
```

The interesting one is not `getName()` (as we understand now, it does as expected) but `getNameExpression()`. This method returns null (which is okay), but while the method works and has a return value (null), it does not get scanned somehow by the visitor. It looks as if the visitor skips the output of methods that return null. 

## I am getting it

This is the code of the `scan` method in TreeScanner:

```
    public R scan(Tree tree, P p) {
        return (tree == null) ? null : tree.accept(this, p);
    }
```

Two conclusions:

- If a method returns null (which is the case for `getNameExpression()`) no visit of the tree will take place
- The scan method needs Tree as argument and cannot handle Name. Therefore, any method returning a Name type cannot be part of the visitor

This also explains why, when a MemberSelectTree is visited, the identifier is not being visited. The method responsible for it is:

```
Name getIdentifier();
```

`scan(Tree tree)` cannot handle it, and thus the visit method for MemberSelectTree does not include the `getIdentifier()` method. Only `getExpression()` is included:

```
    @Override
    public R visitMemberSelect(MemberSelectTree node, P p) {
        return scan(node.getExpression(), p);
    }
```

## IdentifierTree

If an name/identifier is not part of a memberSelectTree or a VariableTree, it is identified as an IdentifierTree. IdentifierTree has this code:

```
public interface IdentifierTree extends ExpressionTree {

    Name getName();
}
```

It's only method returns a Name object, which cannot be an argument to the `scan(Tree tree)` method. We thus expect nothing to be scanned in its visitor method and this is indeed what we see:

```
    @Override
    public R visitIdentifier(IdentifierTree node, P p) {
        return null;
    }
```

## Symbol

It might be a general rule that every Tree with a Name object attached to it also has a corresponding `sym` field attached to it in its JCTree implementation. So while TreeScanner/TreePathScanner won't give you the name, you can just move on to the semantic universe and do everything you want. 

Note: had a chat with ChatGPT and it told me that not every Tree with a Name field has automatically a Symbol field. It also directed me to an interesting method in TreeInfo:

```
    /** If this tree is an identifier or a field, return its symbol,
     *  otherwise return null.
     */
    public static Symbol symbol(JCTree tree) {
        tree = skipParens(tree);
        switch (tree.getTag()) {
        case IDENT:
            return ((JCIdent) tree).sym;
        case SELECT:
            return ((JCFieldAccess) tree).sym;
        case TYPEAPPLY:
            return symbol(((JCTypeApply) tree).clazz);
        case ANNOTATED_TYPE:
            return symbol(((JCAnnotatedType) tree).underlyingType);
        case REFERENCE:
            return ((JCMemberReference) tree).sym;
        case CLASSDEF:
            return ((JCClassDecl) tree).sym;
        default:
            return null;
        }
    }
```

This method provides the Symbol object attached to specific Tree types, but I wonder why it doesn't handle every Tree type that has a Symbol attached. JCVariableDecl definitely has a Symbol attached but it would return null in this method. ChatGPT tells me that this method has a specific purpose, namely finding of references, not declarations. That sounds reasonable.

  
