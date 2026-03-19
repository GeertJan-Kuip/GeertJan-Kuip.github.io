# Analyzing the content of a BlockTree

To do code analysis I want to be able to retrieve the list of StatementTrees from a method body and then see how each statement affects state throughout the application. In this blog post I will provide and discuss the code required to get this analysis right.

## Class and Symbol

To analyze all dependencies in a file, ChatGPT advised me to get my hands on the ClassSymbol object. This is the appropriate code for doing that:

```
public class MyScanner extends TreePathScanner<Void,Void> {

    ClassSymbol currentClass = null;

    @Override
    public Void visitClass(ClassTree node, Void p) {
        currentClass = (ClassSymbol) trees.getElement(getCurrentPath());
        return super.visitClass(node, p);
    }
}

// code to call this, from some other method
MyScanner myScanner = new MyScanner(trees);  // trees is the Trees object
myScanner.scan(cu, null);  // cu is the CompilationUnit object
ClassSymbol currentClass = myScanner.currentClass;
```

After having retrieved the ClassSymbol object (it might be necessary to check if the currentClass variable is really the top class, not some inner class) you can use it to get the symbols.

### Retrieving extends and implements

This is code to get the extends and implements classes/interfaces. The `addDependency` method is defined elsewhere, it collects UML data. Note that this code relies on Type, and that the only required input is the `currentClass` variable:

```
Type superType = currentClass.getSuperclass();
if (superType.tsym instanceof ClassSymbol cs) {
    addDependency(currentClass, cs, EXTENDS);
}

for (Type iface : currentClass.getInterfaces()) {
    addDependency(currentClass, iface.tsym, IMPLEMENTS);
}
```

### Retrieving fields

Again the `currentClass` variable is the relevant input. Note the use of ElementKind, which is an enum living in the not-hidden `javax.lang.model.element` package. I noted that while in this ChatGPT code snippet a check is made on ElementKind.FIELD, it is also possible to call the `isField()` method which is part of this enum class.

```
for (Symbol member : currentClass.getEnclosedElements()) {
    if (member.getKind() == ElementKind.FIELD) {
        VarSymbol field = (VarSymbol) member;
        addDependency(
            currentClass,
            field.type.tsym,
            FIELD_TYPE
        );
    }
}
```

### Retrieving method signatures

Here the method signatures are being retrieved with the `getReturnType()` and `getParameters()`. These methods are declared in interface `ExecutableElement` and are implemented in `public static class MethodSymbol`. They return Type objects, which is why the `tsym` field is being called (this code retrieves Symbol, not Type objects).

```
// Note: this code lives in the same iteration as the previous snippet
if (member.getKind() == ElementKind.METHOD) {
    MethodSymbol m = (MethodSymbol) member;

    // return type
    addDependency(currentClass, m.getReturnType().tsym, METHOD_RETURN);

    // parameters
    for (VarSymbol param : m.getParameters()) {
        addDependency(currentClass, param.type.tsym, METHOD_PARAMETER);
    }
}
```

### Why Type is involved

I asked ChatGPT why this construct was being used:

```
param.type.tsym
```

My initial thougt was that instead of retrieving the type of the symbol and then the symbol of that type, you could just use the symbol itself (`param`). This is wrong, as `param.type.tsym` is actually something different. Let's say you have `String s` as argument in your method signature. The Symbol here is s, but that is not what you are interested in. You are interested in `String`, in the form of a Symbol. Therefore you must go via Type. This pattern, retrieving the Symbol of the Type, is required when you want to get the uml dependencies.

Btw instead of accessing the `type` field directly you can also use `asType()`. This method is declared in the Element interface and returns a TypeMirror object (or a Type object in the implementation of Symbol). Btw TypeMirror is a non-secret interface implemented by Type.

## Dependencies within method bodies

I'm interested in following execution trails that start in methods. This is a step further than my initial ambition, namely the generation of UML data. The chapter below is code that gets uml data from method bodies, after that I'll go deeper into the problem of the 'trails'.

### What I got from ChatGPT about UML data

To get into the bodies of methods you cannot directly call methods on ClassSymbol. Instead, you need to get hold of the current MethodSymbol and work from there:

```
@Override
public Void visitMethod(MethodTree node, Void p) {
    currentMethod = (MethodSymbol) trees.getElement(getCurrentPath());
    return super.visitMethod(node, p);
}
```

Having the currentMethod variable lets you do things like below, to extract the Symbols that represent the type of variables, of new object instances, :

```
@Override
public Void visitVariable(VariableTree node, Void p) {
    VarSymbol var = (VarSymbol) trees.getElement(getCurrentPath());

    if (var.owner == currentMethod) {
        addDependency(
            currentClass,
            var.type.tsym,
            LOCAL_VARIABLE
        );
    }
    return super.visitVariable(node, p);
}

@Override
public Void visitNewClass(NewClassTree node, Void p) {
    Type type = (Type) trees.getTypeMirror(getCurrentPath());
    addDependency(currentClass, type.tsym, OBJECT_CREATION);
    return super.visitNewClass(node, p);
}

@Override
public Void visitMethodInvocation(MethodInvocationTree node, Void p) {
    MethodSymbol called =
        (MethodSymbol) trees.getElement(getCurrentPath());

    addDependency(
        currentClass,
        called.owner,
        METHOD_CALL
    );
    return super.visitMethodInvocation(node, p);
}

@Override
public Void visitMemberSelect(MemberSelectTree node, Void p) {
    Symbol sym = (Symbol) trees.getElement(getCurrentPath());

    if (sym != null && sym.owner instanceof ClassSymbol cs) {
        addDependency(currentClass, cs, STATIC_ACCESS);
    }
    return super.visitMemberSelect(node, p);
}
```

My analysis is that you might be able to do without the `var.owner == currentMethod` condition. The AST is walked depth-first, which means that after any method tree is visited, every tree that is under that method (including all code in the block) is being visited and only then a next method is being visited. The code provided to me by ChatGPT was explicitly meant to retrieve dependencies of a class as used in UML, not to ascribe those dependencies to specific methods.

### Working with MethodTree

I think what I should do is to extract the MethodTree objects within the CompilationUnit object and use visit/scan on each of them to get the startpoints of trails that lead to other parts of the code. The way to do this is to collect the relevant symbols in the method body, with code as in the previous paragraph, and then use this method to find the relevant Tree objects:

```
JCTree tree = (JCTree) trees.getTree(sym);
or 
Tree methodTree = trees.getTree(sym);
or
JCTree.JCMethodDecl decl = (JCTree.JCMethodDecl) trees.getTree(sym);
```

The latter is most versatile. Now that the Tree of the method is obtained, this can be done:

```
JCBlock body = decl.body;
or 
BlockTree body = decl.getBody();
```

The latter of this methods returns a non-forbidden interface object. In both cases you immediately have the correct tree, namely the body of the method, which is a startpoint for a next round of finding symbols. 

ChatGPT warned me for a few scenario's, namely one in which trees.getTree(sym) returns null (happens if the .java file is not available) and one in which the call stack is cyclic. For the latter, I was advised to store the calls and to end the process once a duplicate call emerged.





