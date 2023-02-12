# jvm-ast-encoding-rpn
James Gosling and the JVM; Encoding the Abstract Syntax Tree in Reverse Polish Notation. 

## The Bacground of this

I was whatching Lex Fridman's interview with James, and he said: https://www.youtube.com/watch?v=ZtQRQBeSajo&t=51s

> So And what is it? So the Java virtual machine, you can think of it in different ways. Because it was carefully designed to have different ways of viewing it, so one view of it that most people don't really realize is there is that you can view it as sort of an encoding of the abstract syntax tree in reverse Polish notation. I don't know if that makes any sense at all. I could explain it and that would blow all of our time. But the other way to think of it and the way that it ends up being explained is that it's it's like the the instruction set of an abstract machine that's designed such that you can translate that abstract machine to a physical machine. And the reason that that's important. So if you win back to the early 90s when we were talking to all of these these companies doing consumer electronics.

I was pondering what this was, so I devided to look into into a little better and to reconder thos this all worked, I tried to recreate the AST from several bits. 

So I took this sample code from Rosetta Code: https://rosettacode.org/wiki/Fibonacci_sequence#Iterative_44
```java
public static long itFibN(int n) {
    if (n < 2) {
        return n;
    }
    long ans = 0;
    long n1 = 0;
    long n2 = 1;
    for (n--; n > 0; n--) {
        ans = n1 + n2;
        n1 = n2;
        n2 = ans;
    }
    return ans;
}
```

For this code, I then leveraged the http://lab.antlr.org tool to make sense of the AST, I thought it would give be a better result. I encourage you to have a look and paste the code example, it gave me a great token result, but not a good AST. 
![ This is an Image ](https://github.com/jemo07/jvm-ast-encoding-rpn/blob/c63dc9d81ce99d012b1177d098ad89b65e6c92d3/AEDB9B52-995A-4E07-B9E9-054DFF8123DF.png)
So I just took the code and created a simple text of it. 

```  
Start 
|
return
|
If (IfStatement)
|
< >
|
n (variable)
|
2
|
ExpressionStatement
|
=
|
ans (variable) 
|
+
|
n1 (variable)
|
n2 (variable)
|
Expression
|
=
|
n1 (variable) 
|
n2 (variable)
|
Expression
|
=
|
n2 (variable)
|
ans (variable)
|
For (ForStatement) 
```
Conversely, this would be the Reverse Polish Notation of this AST.
```
n 2 < n n-- 1 > n-- [ n1 n2 + n2 n1 = ans n2 = ] ans return
```
Thus, the execution of this code in the JVM's which is a stack-based architecture, the JVM would perform the following operations:

- Load the value of ***n*** onto the stack.
- Load the constant value ***2*** onto the stack.
- Pop the top two values (***2**** and ***n***) from the stack and compare them.
- If ***n*** is less than ***2***, jump to the return statement.
- Load the constant value ***0*** onto the stack and store it in the local variable ***ans***.
- Load the constant value ***0*** onto the stack and store it in the local variable ***n1***.
- Load the constant value ***1*** onto the stack and store it in the local variable ***n2***.
- Repeat the loop until ***n*** is ***0***:
- Load the value of ***n1*** onto the stack.
- Load the value of ***n2*** onto the stack.
- Pop the top two values (***n2*** and ***n1***) from the stack, add them, and push the result back onto the stack.
- Store the result in the local variable ***ans***.
- Store the value of ***n2*** in ***n1***.
- Store the value of ans in ***n2***.
- Decrement the value of ***n***.
- Load the value of ans onto the stack.
- Return the value on the top of the stack.

So, looking that this, I then took the Fibonacci code in Forth:

```forth
: fib ( n -- fib )
  0 1 rot 0 ?do  over + swap  loop drop ;
```
I re-created the AST for this code as: 
```
Start
|
Drop
|
Loop
|
Swap
|
+
|
Over
|
0 ?Do
|
0
|
Rot
|
1
|
0
```

And attempted to recreate the Java Code based on this AST, please consider that Java uses an array to store the intermediate Fibonacci numbers, rather than using a stack as in the Forth code. For the ***if*** statement checks if the input ***n*** is less than ***2***, and if so, returns ***n*** as the result. I created a loop to calculate the next Fibonacci number with each iteration, storing the result in the fibs array. The function ends by returning the last Fibonacci number stored in the fibs array.

here is my probably poor attempt: 
```java
static long fib(int n) {
    if (n < 2) {
        return n;
    }
    long[] fibs = new long[n];
    fibs[0] = 0;
    fibs[1] = 1;
    for (int i = 2; i < n; i++) {
        fibs[i] = fibs[i - 1] + fibs[i - 2];
    }
    return fibs[n - 1];
}
```

And I guess it does run, after some iterations to make it into a class, and at first I made it public, but this is not supported. Happy to end this rabitt hole successfully. Here is is the in Complier Explorer : https://godbolt.org/z/Y31Pvneqc 
