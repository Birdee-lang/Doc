# 12. Exception handling

## 12.1 Exception handling basics

Exceptions means that the program enters an uncommon state at the run time. Exceptions may occur when the program has encountered some errors like dereferencing null or dividing by 0:

```vb
dim a as string = null, b as int = 0
println(a) 
dim c = 32/b
```

Birdee programmers can also manually generate an exception in their Birdee code. Let's first consider what the program will be like without exceptions. For example, if we need to write a program that checks the communication with a server. We are given a function "send\_msg" that can send a message to the server. If sending message fails, "send\_msg" returns false. If successful, it returns true. We are required to send 4 messages to the server. If any one of the message fails, we must print "Connection Down". The program will be:

```vb
if send_msg()==false then
	println("Connection Down")
else if send_msg()==false then
	println("Connection Down")
else if send_msg()==false then
	println("Connection Down")
end
```

There are three nested "if-else", making the program less readable. We will see later how exceptions can simplify the program.

To throw an exception, you can create a class object and use the "throw" keyword to raise the exception.

```vb
throw {exception_object}
```

The class of "exception\_object" must be annotated "enable\_rtti" when defined. For example,

```vb
@enable_rtti
class SomeException
	public msg as string
	public __init__(msg as string) => this.msg = msg
end

func sss()
	throw new SomeException("hello")
end
```

To handle an exception, you must put the code that may throw an exception in a "try" block, and put the code that handles the exception in "catch" blocks. The syntax of "try-catch" error handling is:

```vb
try
	...
catch {exception_object_name} as {exception_type}
	...
catch {exception_object_name} as {exception_type}
	...
...
end
```

The code between "try" and the first "catch" is the "try block". The one "try" should be followed by at least one "catch" block. Each catch block has a "exception\_type". Once an exception is thrown in the try block, the type of the throw exception object will be checked against the "exception\_type" of each of the catch blocks in the order of defined in the source code. If a catch block's "exception\_type" is the same class or the super class of the throw exception object, this catch block will be executed. And you can access the thrown exception object with variable "{exception\_object\_name}". If multiple "catch" blocks can match the exception, only the first "catch" block will be executed.

If no catch-block is matched with the thrown exception, the exception will be handled by the current function's caller's try-catch blocks (if there is any). If no match in the caller's function, the exception will be recursively passed to the related try-catch blocks of functions in the call stack, in the direction from the current function to the top-level code. If still no catch-block is matched, the program will be terminated. 

Note that if an exception is throw, the program will jump to the matched catch block, and will not execute the code after the execption. After a catch block is executed, the program will continue its execution at the end of the matched try-catch block.

In the above example of checking server communications, we can improve the program by using exceptions. Now if it fails to send the message, the function "send\_msg" will throw an exception of type "connect\_error".

```vb
try
	send_msg()
	send_msg()
	send_msg()
catch e as connection_error
	println("Connection Down")
end
```

Here is an example that nested try-catch handles one exception:

```
@enable_rtti
class E1
   public dummy as int
   public str as string
   public func __init__(str as string) => this.str=str
   public func print() => println("E1"+str)
end

@enable_rtti
class E2
   public str as string
   public dummy as int
   public func __init__(str as string) => this.str=str
   public func print() => println("E2"+str)
end

@enable_rtti
class E3
   public dummy as int
   public dummy2 as int
   public str as string
   public func __init__(str as string) => this.str=str
   public func print() => println("E3"+str)
end

function f1()

   try
      f2()
      println("try conti")
   catch e as E1
      e.print()
   catch e as E2
      e.print()
   catch e as E3
      e.print()
   end
   println("Continue")
end

function f2()

   try
      throw new E3("yo")
   catch e as E1
      print("f2")
      e.print()
   catch e as E2
      print("f2")
      e.print()
   end
end

f1()
```

This example throws an "E3" in f2 and the function f2 cannot handle the exception. The exception will finally be handled by function "f1". So the output of the program will be:

```
E3yo
Continue
```

## 12.2 Hardware generated exceptions

In the above section, we have shown two kinds of exceptions that can be thrown by the hardware: Memory access exception and Zero Division. You can catch these two exceptions with exception class "mem\_access\_exception" and "zero\_div\_exception", respectively.

```vb
func test()
	dim a as string=null
	println(a)
end


func wrapper1()
	try
		test()
	catch e as mem_access_exception
		println("Caught SEGV")
	end
end

wrapper1()
wrapper1()
wrapper1()

func test2[T]() as T
	dim a as T = 0
	return 2/a
end


func wrapper2[T,STR as string]()
	try
		test2[T]()
	catch e as div_zero_exception
		println(STR)
	end
end

wrapper2[int,"int"]()
wrapper2[int,"int"]()
wrapper2[int,"int"]()
```