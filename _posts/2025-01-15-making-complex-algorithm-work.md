## Making complex algorithm work

I'm writing a Java class that tokenizes .java files. It's a sort of win-win as I'm solving code problems working on a tool that helps me analyze Java code. As of now I don't know much about tweaking performance, but this is how the algorithm works:

- The .java file is processed one line at a time
- Each line is converted to aan array of ASCII character codes
- The array is processed character for character, whereby the index of the start and end of each token are recorded (stored in another array)
- to make it work, a 'mode' variable is introduced. Depending on the previous character, the mode is set (values 0-6). This allows for context when interpretating the character.

The algorithm is a long list of nested if/if else/else statements. The problem is to solve the bugs, as there are always mistakes in it on first try. My strategie for debugging is the following:

- use the debugger
- total rewrite after careful evaluation, to get clearer code
- use separate methods with clear names so the code becomes better readable. For example I created the method 'isDoubleQuote' to find double quote, and 'put' to write an index to the dedicated 'positions' array
- have a strategy that makes it fast and easy to compare expected results with real results

Hopefully I manage to make it work, and hopefully I find good strategies to get rid of the bugs. It's the latter that matters most.