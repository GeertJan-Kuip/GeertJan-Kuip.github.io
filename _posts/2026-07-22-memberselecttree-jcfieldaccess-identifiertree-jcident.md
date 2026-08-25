# MemberSelectTree, JCFieldAccess, IdentifierTree, JCIdent

I thought I had understood enough of the Abstract Syntax Tree to be comfortably able to get any information form method bodies that I would need. It turns out there are some things I need to dive a bit deeper into.

What I wanted to have is the opportunity to tell for every member field of a class, where and how it is touched by the code in the methods of that class. 

## What Intellij has

In Intellij, if you Ctrl+click a member field, you get a popup with 'usages'. It is a list of all places in the code that read or write to this variable, and you can choose to see only one of them. It is also possible to limit the read/write access list to the current file, to files within the module, to open files etc. 

Basically I want to be able to gather the same sort of information that Intellij provides. My application must subsequently be able to explain, in text, what the character of the field is. It might be 'effectively final', 'only accessed from within' or 'effectively private', 'mutated but only by one class', 'mutated multiple times but in different ways' etc.

## Finding the occurences

It starts with finding places in the code where the identifier of the member field is used. As, having used 'analyze()' during the compilation process, all symbols have been resolved, this is a search for the right symbol, not the right name. You do not have to figure out whether things with equal names refer to the same variable, this is already done by the compiler.

You need to extend TreePathScanner to be able to visit all trees in the AST and the two types you need to override are `VisitMemberSelectTree` and `VisitIdentifierTree`.

### VisitMemberSelectTree

This method deals with dot-separated expressions. Think of `this.someMemberFieldName.value1`. 

- more to come

