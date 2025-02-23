

Knowing a lot about the [language specification](https://docs.oracle.com/javase/specs/index.html) helps when answering test questions. The [new book](https://www.amazon.com/Oracle-Certified-Professional-Developer-Complete/dp/1119619130/ref=sr_1_1) I bought strongly advised to check first if the code in a question would actually compile. If not, you can select the 'does not compile' answer and proceed. Not compiling can be the result of minor syntax things like missing semicolons. The Java language specification is sort of intimidating but reading it does provide new insights.

#### The unary + operator

The unary + (or -) operator promotes the primitives types of byte, short and character to int. It promotes the reference types of Byte, Short and Character to Integer. This is relatively unimportant for primitive types. The code snippet below both work fine:

```
byte a = 3;
int b = +a; // +a returns integer value 3, this value is assigned to b.
int c = a;  // automatic type promotion, net effect is the same as line above.
```

For boxed numbers the effect is more substantial:

```
Byte a = 3;
Int b = a;  // doesn't compile. Boxed numbers behave awfully.
Int b = +a; // this does compile because ot the unary operator that converts a to a primitive int.
```

#### Type conversions, castings and characters

Type castings work remarkably well with character primitives, I wasn't so aware of it. The sample below has a stream of primitive ints:

```
IntStream.rangeClosed(65, 72).forEach(x->System.out.printf("%s | ", (char)x)); 

// prints A | B | C | D | E | F | G | H |
```

With boxed numbers it's again a bit more complicated. The code below has a stream of Integer reference types. The '+' can help you out, converting Integer to int:

```
List.of(65,66,67,68,69,70,71,72).forEach(x->System.out.printf("%s | ", (char)(x)));  // doesn't compile, cannot cast Integer to char

List.of(65,66,67,68,69,70,71,72).forEach(x->System.out.printf("%s | ", (char)(+x))); // this one works because of the +

// prints A | B | C | D | E | F | G | H |
```





