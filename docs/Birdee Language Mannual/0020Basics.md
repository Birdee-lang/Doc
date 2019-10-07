# 2. Language Basics

In this section, the basic syntax of Birdee will be explained, and building more complex programs become possible.

## 2.1 Variables and expressions

Variables are holders of data. They have types specified when they are defined. There are basic types in Birdee, which include boolean (true/false), byte (8-bit signed integer), short (16-bit signed integer), int (32-bit signed integer), long (64-bit signed integer), uint (32-bit unsigned integer), ulong (64-bit unsigned integer), float (32-bit float-point number), double (64-bit float-point number) and pointer (native pointer type). Complex types like array and class will be explained later. (Note: the class type "string" is one of the most commonly used class in Birdee, and we have already used it in our "hello world" example!)

### 2.1.1 Variables definitions 

To define a variable, use the keyword "dim". The basic syntax is shown below. The variable\_name can be any valid name composed by "\_", numbers (0-9) and English characters (a-z and A-Z). In addition, the variable\_name should not be one of the keywords of Birdee, such as "dim" and "as". The type can be a basic type, array type or class type.

```vb
dim {variable_name} as {type}
```

Below is an example to define a variable named "v" with integer type in the top-level scope.

```vb
dim v as int
```

You can assign a value to the variable when defining it.

```vb
dim v as int = 123
dim str as string = "hello"
```

You can define multiple variables in one line with only one "dim" keyword, separated by comma. (and you can still use "=" to assign an initial value). 

```vb
dim v as int = 123, str as string = "hello", length as float
```

The type can be omitted and inferred by the compiler in a variable definition when the definition has an initial value assigned. The above example code can be simplified to:

```vb
dim v = 123, str = "hello", length as float
```

Note that the variable "length" has no initial value assigned, so the type must be given.

### 2.1.2 Variable assignment

To use a variable, you should first define it with a specific type (or assign an initial value to it). For example,

```vb
dim v1 as int, v2 as float, v3 as string, v4 as int
```

You can assign values to variables by "=":

```vb
v1=123
v2=3.14
v3="birdee"
```

Note that in the above code, the value "birdee", which is surrounded by a pair of quotation marks ("), is a string literal. The string literals will be explained later.

Variable does not allow to be assigned with incompatable types. For example, the following two lines of code are not allowed by Birdee.

```vb
v1="123"
v3=321
```

It is because "v1" is of int type, which cannot be assigned a string value. And "v3" is a string variable, and does not accept integers.

### 2.1.3 Operators

Birdee allows arithmetic(+,-,\*,/,%, etc.) and logical operators (&, && ,| , ||, etc.) in expressions. The meanings and precedence of these operators are shown below:


|  Operator |  Meaning |  Precedence |
|---|---|---|
|  * |  Multiplication  |  15 |
|  / |  Division |  15 |
|  % |  Remainder |  15 |
|  + |  Addition |  14 |
|  - |  Substraction |  14 |
|  < |  Less than |  11 |
|  > |  Larger than |  11 |
|  >= |  Larger than or equal |  11 |
|  <= |  Less than or equal |  11 |
|  == |  Value equal |  10 |
|  != |  Value not equal |  10 |
|  === |  Reference equal (for class objects/arrays) |  10 |
|  !== |  Reference not equal (for class objects/arrays) |  10 |
|  & |  Bitwise and |  9 |
|  ^ |  Bitwise xor |  8 |
|  \| |  Bitwise or |  7 |
|  && |  Logical and |  6 |
|  \|\| |  Logical or |  5 |
|  = |  Assignment |  4 |

You can use brackets "()" to change the default precedence.

TBD: explain Logical AND and OR

For value types (integers, floats, pointers, ...), the operators "==" and "===" are equivalent (so are "!=" and "!=="). For references (class objects and arrays), the operators "==" and "!=" call the methods "__eq__" and "__ne__" of the left hand side object and use the right hand side object as the parameter, which compare whether the value of objects are equal. The operators "===" and "!==" only compares the references themselves.

For now, we can write a bit more complex programs:

```vb
dim v1 as int, v1 as int, v3 as float
v1 = (12345 + 999) * 6789
v2 = v1 % 32
dim v4 as boolean = v2 > v1
println(int2str(v2))
println(bool2str(v4))
```

Note: "println", "bool2str" and "int2str" are system functions. "println" will print a line of string on the console (we have already used it in our "hello world"). "bool2str" and "int2str" are functions that convert boolean and int values to strings, respectively.

### 2.1.4 Operators for class types

The operators in the above section are originally for basic types like int, float and boolean. However, Birdee allow you to apply operators on class types. We will explain it later, but we will now introduce an operator for our old friend - the string class, which is widely used. You can add (+) two strings by the operator "+" to concatenate one after the other:

```vb
dim v1 = "birdee"
dim v2 as string
v2= v1 + " is awesome"
println(v2)
```

The above funtion concatenate strings "birdee" and " is awesome", then prints out the result string "birdee is awesome".

### 2.1.5 Auto type conversion

If the types of the operands of an operator do not match, and the operands are both of numeric types (int, uint, float, etc.), Birdee compiler will try to do auto conversion. In an operator expression, the type with a smaller "promotion value" will be converted to the type with a larger "promotion value", and the resulting type of the expression will be the larger "promotion value" type. The promotion values for numeric types are listed below:

|  Type |  Promotion values |
|---|---|
|byte|-2|
|short|-2|
|int|0|
|uint|1|
|long|2|
|ulong|3|
|float|4|
|double|5|

For example, 

```vb
dim a as byte = 1, dim b as long =3
b=a+b
dim c as float=0.1
dim d = c+b
```

The above program adds a byte (a) with a long (b). It will first convert the value of "a" to type long, and add with "b", resulting in a type long value. Similarly, when adding "c" with "b", it will convert "b" to type float and the result is in type float, making variable "d" a float variable.

A functype value can be automatically converted to closures (functype and closures will be discussed later). A value of a subclass can be automatically converted to a sub-class reference (classes will be discussed later).

Important note: The assignment operator will always try to convert the value on the right of "=" (we call it Rvalue) to the variable on the left of "=" (called Lvalue). If type of Rvalue cannot be converted to Lvalue, an error will be raised. 

### 2.1.6 String literals

We have met string literals in above examples. Characters surrounded by a pair of quotation marks (") represents a string literal, Birdee also supports escape characters like "\\n", "\\\\".

Also, Birdee supports raw strings, which will disregard any escape characters. A raw string starts and ends with three quotation marks ('''). You can even have multiple lines in a raw string.

```vb
println("Hey\nThis is Birdee!")
println('''Hey
This is Birdee!''')
println('''Hey\nThis is Birdee!''')
```

The above program will output:

```vb
Hey
This is Birdee!
Hey
This is Birdee!
Hey\nThis is Birdee!
```

Note that the first two string literals are equivalent. The third string literal does not parse the escape character "\\n".

TBD: introduce other constants

## 2.2 Comment

Comments in the source code could help you and other programmers better understand the program. The Birdee compiler will ignore the comments in a program when it is compiled. One way to write comments is to use "#", and it will let the compiler to ignore "#" and later characters in the source code until the end of the current line.

```vb
dim a as int = 3+4  # "a" is an integer with value 7
println(int2str(a)) # print the value of "a"
```

To write comments in multiple lines, you should use "##" to mark the start of the comment and use another "##" to mark the end of the comment.

```vb
## This program computes the value of 3+4,
and then put the value in the variable "a".
It finally print the value of "a" ##
dim a as int = 3+4  
println(int2str(a))
```

## 2.3 Function

A function in Birdee is a reusable block of code. To define a function in the top-level, use syntax like:

```vb
function {function_name} ([ {parameter1} as {type}, {parameter2} as {type}, ... ]) [as  {type}]
	...
end [function]
```

In the example, "{ }" means names that you must define and "[ ]" means optional syntax which can be omitted.

A function with a returning value should specify the returning type at the end of function definition ("[as  {type}]"), but a function without returning value can omit it. A function can have zero or multiple parameters, with each parameter defined by "{parameter} as {type}" in the function definition. The "end" in the last line of the function definition marks the end of the function. To make Birdee look similar with the syntax of Visual Basic, you can write an additional (and optional) "function" word after "end", and the "function" word will be omitted by the compiler.

We have already met some functions like "println" and "int2str". They are system functions provided and defined by Birdee.

We first show an example of a function doing addition:

```vb
function add (a as int, b as int) as int
	dim c = a + b
	return c
end

dim result=add(3,4)
```

The function "add" takes two parameters: "a" and "b", both of integer type and returns an integer. In the body of the function, it adds the two parameters and return the result by the "return" keyword. In the last line, we call the funtion, and save the result in a variable. Note that unlike the code in the top-level, the code in a function will not be executed unless the function is called.

To call a function, type the funtion name followed by a pair of brackets, with the parameters filled in order. If the funtion returns a value, you can use the function call expression as a value:

```vb
println(int2str(12345))
```

In the above example, "int2str(12345)" is a function call expression. The function "int2str" takes the integer "12345" as the input and returns a string format of it. So the expression "int2str(12345)" itself is a string, and is used as the input of the function "println", to print the string in the console.

There is a simpler way to define a function in a single line. The syntax is:

```vb
function [function_name] ([ {parameter1} as {type}, {parameter2} as {type}, ... ]) [as  {type}] => {expression}
```

You can use the keyword "=>" to define the function's body in a single expression. Note that if the function has a return type, the expression of the function body will be automatically returned. You can even omit the function name in function definition. This is useful for functional programming. In addition, the keyword "function" can be simplified to "func". So a shorter form of the "add" function in the last example can be:

```vb
func add(a as int, b as int) as int => a+b
```