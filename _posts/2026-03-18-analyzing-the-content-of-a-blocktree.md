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

##




