## Declaring and initializing arrays 

Here my notes on simple arrays. They lack the methods that higher order collections have but you can use static methods from the Arrays class to compensate for it. You can be creative when initializing an array, uninitialized variables are not permitted, and it's okay to use them to store the children of interfaces or abstract classes.


### On valid ways of initializing

All of the following statements compile:

```
int  []myArray1 = new int[3];
int[]  myArray2 = new int[] {1,2,3};
int myArray3  [] = {1,2,3};

int[][] myMultiArray1 = new int[2][2];
int  []myMultiArray2 [] = new int[2][];
int myMultiArray3[][] = {{1,2},{3,4}};
int[]  myMultiArray4[] = new int[8%3][3-1];
```


