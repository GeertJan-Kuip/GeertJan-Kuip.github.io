# Variable naming and method understanding

This is just a short blog post, I was thinking about ways to easier understand methods. 

## The problem

If you look at a random method you see a bunch of variable names. Those names often try to tell you what they represent but that does not mean that you will grasp things immediately. Hoovering over an identifier gives you information about its type and with Ctrl-Click you can see its source file.

## The idea

Would it be handy if variable names are just the type they represent? It means you can more easily see which identifier represents which type and therefore you can easier grasp eventual methods or field access called on the variable. Let's have two pieces of code, one in 'normal' style and one in this new, somewhat exagarated style (I'm using capitals):

```
    static void method2(Iterable<? extends CompilationUnitTree> cuTrees, Trees trees) {

        String indent = "   ";
        ScannerE scannerE = new ScannerE(trees);
        for (CompilationUnitTree cu : cuTrees){

            scannerE.scan(cu, null);
        }

        for (Symbol.ClassSymbol e : scannerE.listClassSymbol){
            System.out.println(e.fullname);
            for (Symbol member : e.getEnclosedElements()) {
                if (member.getKind() == ElementKind.FIELD) {
                    Symbol.VarSymbol field = (Symbol.VarSymbol) member;
                    System.out.println(indent + field);
                }
            }
            System.out.println();
        }

    }
```

```
    static void method2(Iterable<? extends CompilationUnitTree> LIST_COMPILATIONUNITTREES, Trees TREES) {

        String STRING_1 = "   ";
        ScannerE SCANNER_E = new ScannerE(TREES);
        for (CompilationUnitTree COMPILATIONUNITTREE : LIST_COMPILATIONUNITTREES){

            SCANNER_E.scan(COMPILATIONUNITTREE, null);
        }

        for (Symbol.ClassSymbol SYMBOL_DOT_CLASSSYMBOL : SCANNER_E.listClassSymbol){
            System.out.println(SYMBOL_DOT_CLASSSYMBOL.fullname);
            for (Symbol SYMBOL : SYMBOL_DOT_CLASSSYMBOL.getEnclosedElements()) {
                if (SYMBOL.getKind() == ElementKind.FIELD) {
                    Symbol.VarSymbol SYMBOL_DOT_VARSYMBOL = (Symbol.VarSymbol) SYMBOL;
                    System.out.println(STRING_1 + SYMBOL_DOT_VARSYMBOL);
                }
            }
            System.out.println();
        }

    }
```



