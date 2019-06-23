# 3. Flow Control

The syntax introduced till now allows programs to be executed sequentially. But how to let different parts of the program be executed under different conditions? How to execute some blocks of code for multiple times? The answer is by flow control!

## 3.1 Conditional Branches

### 3.1.1 If

The "if" syntax is used to execute a block of code when the condition is true. The basic syntax is:

```vb
if {condition} then
	...
end [if]
```

The "{condition}" should be an boolean expression (could be true or false), such as "a>0", "a!=b && b>32" and so on. The code within an "if" block will be executed if the condition is true. To mark the end of the block of code, use "end" or "end if". No matter the condition is true or not, the program will continue at the line after "end" or "end if", after the "if" block is executed. The following example checks if the value of "a" is larger than 100. If so, it prints a line "a is larger than 100". The program will always print the value of "a" no matter what value "a" is.

```vb
if a>100 then
	println("a is larger than 100")
end
println("a is " + int2str(a))
```

You may want to wirte a program like this: if a condition is true, execute block A; otherwise, execute block B. You can achieve this by the "if...else..." syntax. For example, we could change the above program a little bit: if a is not larger than 100, we also want to print a line "a is no larger than 100":

```vb
if a>100 then
	println("a is larger than 100")
else
	println("a is no larger than 100")
end
println("a is " + int2str(a))
```

The "else" key word will mark the end of the block to be executed if true, and mark the start of the block to be executed if false.

In many cases, multiple conditions should be considered. For example, a teacher may want to find the grade from a score. The grades are given according to the following table (assume the scores are integers):

|  Grade |  Score |
|---|---|
|A| 90-100|
|B|75-89|
|C|60-74|
|F|0-59|

We can write a Birdee program to solve the problem:

```vb
function find_grade(score as int) as string
	if score<=59 then
		return "F"
	else
		if score<=74 then
			return "C"
		else
			if score<=89 then
				return "B"
			else
				return "A"
			end
		end
	end
end
```

Yes, "if" blocks can be nested, but the above program is not an elegant way to solve this program. The nested "if...else..." blocks can be simplified by "else if" blocks:

```vb
function find_grade(score as int) as string
	if score<=59 then
		return "F"
	else if score<=74 then
		return "C"
	else if score<=89 then
		return "B"
	else
		return "A"
	end
end
```

The "else if" syntax saves some "end"s and make nested "if" blocks looks better.

## 3.2 Loop

### 3.2.1 For Loop

To execute a block of code for a specific times, "for loop" can be used. The basic syntax is:

```vb
for {variable} = {start} to {end}
	...
end [for]
```

The "{variable}" should be any Lvalue in integer type. {start} and {end} are any valid integer expressions. The {variable} will increment from {start} to {end} and the block of code in the loop will be executed for ({end} - {start} + 1) times. If {end} < {start}, the loop stop executing. The following code will print "hi! From i=1", "hi! From i=2", "hi! From i=3".

```vb
dim i as int
for i=1 to 3
	println("hi! From i=" + int2str(i))
end
```

The variable "i" is called loop variable, which is modified by Birdee language every time the body of the loop is executed. Note that internally, Birdee will generate code to increment the loop variable at the end of the loop body and compare the variable against the {end} expression to decide whether to run the loop body again.

You can combine the definition of loop variable with the for loop. For example, you can simply modify the above example to:

```vb
for dim i=1 to 3
	println("hi! From i=" + int2str(i))
end
```

However, the loop variables are defined in different scopes in the previous two examples. (What is "scope"? We will explain later.) Keep in mind that if you define a variable in a for loop (in the for ... to ..., or in the body of the loop), the variable can only be used within this particular for loop and is not available outside of it. For example, we add a line in the above example:

```vb
for dim i=1 to 3
	println("hi! From i=" + int2str(i))
end
println(i+1)  # Compiler will complain here!
```


Birdee compiler will recognize an error in the last line because variable "i" is only defined within the for loop.

Also note that {start} and {end} are not necessarily constants, they can be variables, function calls or whatever valid integer expressions.

If you do not want to include the {end} value in the for loop, you can use "till" key word instead of "to" key word:

```vb
for dim i=1 till 3
	println("hi! From i=" + int2str(i))
end
```

The above for loop will be executed twice, with i=1 and then i=2.

You may want to jump out of the loop in some cases, where you can use the "break" keyword to jump to the next line of the end of the loop:

```vb
for dim i=1 to 1000
	println("hi! From i=" + int2str(i))
	if i==2 then
		break
	end
end
println("End of the loop")
```

The "break" keyword in the above program will let it jump to the line "println("End of the loop")". Then you will see two lines with i=1 and i=2 printed in console and a line "End of the loop", after the program is executed.

You can use "continue" keyword to skip the rest of the loop body in the current iteration, and continue at the next iteration:

```vb
for dim i=1 to 3
	if i==2 then
		continue
	end
	println("hi! From i=" + int2str(i))
end
```

The above example will print 2 lines with i=1 and i=3. Because when i=2, the loop is skipped.

You cannot use "break" and "continue" outside of loops.

### 3.2.2 While Loop

The "while" loop will execute a block of code repeatedly until the given condition becomes "false". The syntax of it is:

```vb
while {condition}
   ...
end [while]
```

The "condition" should be a boolean-typed expression. Here we present an example for printing strings from a "black box". Assume that the "black box" contains some strings and you can call "get" function to fetch a string from the box, and you can call "has_next" function (which returns a boolean value) to find whether there are any strings in the black box. 

```vb
while has_next()
    println(get())
end
```