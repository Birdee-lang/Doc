# 11. Functional Programming

## 11.1 Function variables

Functions of Birdee are expressions. You can assign a function to a variable and use the variable to call the function. For example,

```vb
dim func_var = function (a as int, b as int) as int => a+b
println(int2str(func_var(1,2))) 
```

The first line defines a "one line" function, which returns the sum of the arguments. The function definition begins from the keyword "function". Note that we can define a function without naming it, which results in an anonymous function. The function definition itself is an expression, and we assign it to a variable "func\_var". Then in the second line, we use this variable as if it is a function.

The function variable is "mutable", which means you can re-assign another function to it.

```vb
dim func_var = function (a as int, b as int) as int => a+b
println(int2str(func_var(1,2)))

func_var = function (a as int, b as int) as int => a-b
println(int2str(func_var(1,2)))
```

What's the type of a function variable (e.g. "func\_var" in the example)? Birdee introduces a new type system called "functype". You can define a "functype" by:

```vb
functype {typename} ([arguments...]) [as {return_type}]
```

It looks like declaring a function, but it actually defines a type. A "functype" defines a type of functions, with the given prototype (the argument types and return type). After using "functype" definition, you can define a function variable without initializing it with an existing function. For example,

```vb
functype myfunctype (v1 as string, v2 as int)

dim func1 as myfunctype, func2 as myfunctype
func1 = func (str as string, i as int) => println(str + int2str(i))
func2 = func1
func2("My student id is ", 23)
```

In the example, we define a functype called myfunctype. We define two function variables "func1" and "func2". We assign a function to func1 and assign func2 with the value of func1. Finally, we call the anonymous function with the variable func2.

A function variable can be assigned with a function definition, a named function or another function variable, as long as the prototype of them are the same. Birdee considers two prototypes are the same if and only if the order and the types of the arguments are the same, and they have the same return type. Note that the functype name, the function name and the argument names are not considered when comparing the prototypes.

```vb
function add(a as int, b as int) as int
	return a+b
end

functype thefunctype (v1 as int, v2 as int) as int

dim foo as thefunctype
foo = add #okay

functype otherfunctype (fff as int, bbb as int) as int

dim bar as otherfunctype = foo # okay

foo = func (a as int,b as int) => a+b #fail to compile here!
```

In the above example, the functype "thefunctype" and "otherfunctype" are exactly the same. The last line of code will fail to compile, because the return type of the function (which returns nothing) and the return type of "foo" (which is int) are different.

Sometimes for simplicity, you can use in-place definition of functype without defining it in advance. For example,

```vb
function callfunc(f as functype (v as int), b as int) 
	f(b)
end

function print_int(a as int) => println(int2str(a))

callfunc(print_int,32)
```

The function "callfunc" takes 2 arguments. The first one is a function variable, which has the type "functype (v as int) as int". The "callfunc" function simply calls the function variable "f" with the argument "b". We pass the function "print\_int" as a parameter to the function "callfunc".

## 11.2 Closures

Before talking about closures, we first introduce a feature of Birdee, that you can define functions in the body of other functions. For example,

```vb
function outer()
	dim inner = function (str as string)
		println(str)
	end

	for dim i = 1 to 10
		inner(int2str(i))
	end
end
```

This example defines a "inner" function which can only be called within the "outer" function.

In the inner nested function, you can access the local variables (and the arguments) of the parent or ancestor outer function. For example,

```vb
function outer()
	dim prefix = "i is "
	dim inner = function (str as string)
		println(prefix + str)
	end

	for dim i = 1 to 10
		inner(int2str(i))
	end
end
```

The "inner" function accesses the "outer" function's "prefix" variable. The "inner" function is called a closure, because it "captures" the variable of outer function.

To explain what "closure" is, we first explore how local variables work in a function without being captured. The local variables (or maybe some arguments) of a function are stored on the stack of the program. Once the program enter the function, the variables will be allocated on the stack, and they will be destroyed after the function returns. Without capturing, the life time of local variables is controlled by the function. If a local variable is used in the inner function, the inner function "captures" the variable defined in the parent function. Now the captured variables are owned by both parent function and inner function. An inner function with captured variables is called a "closure".

Closures can be viewed as functions with its internal states. Note that a function without capturing variables has no internal states, because it either uses global variables (not internal), or uses local variables which cannot persist after the function returns. In contrast, a closure uses captured variables to store its local states. Like functype, we can define a closure type by the syntax:

```vb
closure {typename} ([arguments...]) [as {return_type}]
```

The usage of closure type is almost the same as functype. For example, we define and use a closure:

```vb
closure counter_closure () as int
function get_counter() as counter_closure
	dim i as int = 0
	dim f = func () as int
		i = i + 1
		return i
	end
	return f
end

dim counter as counter_closure = get_counter()
println(int2str(counter()))
println(int2str(counter()))
println(int2str(counter()))

counter = get_counter()
println(int2str(counter()))
```

In this example, we define a closure type called "counter\_closure". Then we construct a function that returns a "counter\_closure". It builds a function "f", which captures a local variable "i" of "get\_counter". In function "f", it increases "i" and return the value of "i". The closure function "f" is returned by function "get\_counter". We then define a closure variable "counter" and use "get\_counter" to assign a closure to it. We call "counter" three times. The magic happens here! The closure function "counter" outputs 1, 2, and 3 in three calls to it, which means it remembers its internal states. Then we re-assign a new closure instance to "counter", and it returns 1 after calling it. So the output of this example is "1", "2", "3" and "1". Why? Because when "get\_counter" returns a new closure, it creates a new instance of the internal state (the variable "i"). The closure will bind the internal state with the internal function "f". Calling the closure will modify the captured variable "i" in the state. Another call to "get\_counter" creates another instance of the state.

Closure and functype are quite similar. Closure can be in-place defined too. The only difference is that functype only points to a function without state. You can assign to a closure variable with a functype variable (or a function without captured variable). However, you cannot assign to a functype variable with a closure variable or any function that captures parent variables. Note: using global variables in a function will not capture a global variable.

How are closures implemented in Birdee? When a parent function finds that its local variable is captured by a inner function, it will put the variable in a "internal state object". When the parent function is called, it will allocate the internal state object on the heap. A closure variable contains two values, the first is the address of internal state object and the second is the function pointer. In contrast, a functype variable is just a function pointer in the low level of the compiler. The inner function that captures outer variables will have an additional hidden argument to get the internal state object from the closure variable. The parent function will assign its address of internal state object to the closure. When the program calls a closure, the compiler first fetch the internal state object address and the function address from the closure variable. It then call the function with the object address and some other arguments.

As we can see, the default implementation of closure makes the parent function allocate the captured variables on the heap, which could be a performance concern. Sometimes the inner functions of a function are only used in the parent function, and the inner function closure is never used by any non-ancestor functions, we can allocate the the captured variables on the stack. Only if you are sure about it, you can use the "stack\_capture" annotation on the parent function to make faster closures.

### 11.2.1 Binding objects with methods using closures

Closures can be used to hold a reference to an object with a method. The expression 

```vb
object_expression.member_function
```

can be automatically converted to a closure of the same prototype of the member function. For example, the "string" class has a member function "getRaw", you can bind a string with the function by:

```vb
dim str = "hello"
dim funct as closure () as pointer = str.getRaw # note: no "()" here
funct() # the same effect as "str.getRaw()"
```