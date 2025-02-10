## Operator precedence

In 1Z0-819 questions involving operator precedence are to be expected. This is the order:

```
var++ var--
++var --var +var -var ! ~
* / %
+ -
<= >= < > instanceof
== !=
>> << >>>
| & ^
|| &&
? :
= += -= *= /= %= ^= |= &= <<= >>= >>>=
```

By learning this I encountered some things I didn't know before:

- var++ and ++var differ in their return value
- ~ inverts a sequence of bits (1101 becomes 0010)
- <<, >> and >>> deal with shifts in bit sequences. Useful in low level programming.
- ^ is the XOR operator, not the exponential
- there is no equivalent for ^ among the logical operators, you might use !=
- when using bitwise logical operators, expressions at both ends are evaluated, no matter wether the left expression already determines the outcome. This is not the case with logical operators, those are evaluated from left to right whereby the expression on the right won't be evaluated if it won't have effect on the outcome of the statement
- <<, >>, >>>, <<=, >>=, >>>= are all about working on bit level
- instanceof is a 'relational' operator, the only one using a word
- you need the math library to do exponentiation (Math.pow(base, exponent)). To square a number use value*value.
