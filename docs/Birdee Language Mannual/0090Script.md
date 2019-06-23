# 9. Compile-time scripts 

You can embed Python scripts in Birdee codes. The scripts are executed in compile time and these scripts will NOT be executed in the run time. Generally, the script is used for 

 * generating and compiling Birdee code in compile-time
 * altering the existing Birdee code

Compile-time scripts can completely replace macros and template-based meta-programming. In C++, these two features are powerful but hard to master - that's one of the reason of starting this programming language project.


## 9.1 Script basics

To embed Python scripts in Birdee source code, you only need to surround your Python script with "{@" and "@}". For example

```vb
{@
print("hello world")
@}

dim a = "hey world"
println(a)
```

The above is a Birdee program. It can be divided into two parts. The first part is a Python script, which prints a string "hello world". When the program is compiled, the script will be run by the Birdee compiler, and prints the string in the standard output of the compiler itself. The second part is the Birdee code. It defines a variable and prints a string "hey world". This part will be executed in the run time.

What can embedded Python scripts do? You can use them to generate Birdee code and insert the generated code into the place where "{@" and "@}" is. The "{@" and "@}" with the script can serve as a statement, expression or a type specifier. An expression is a piece of code which represents a value, like "1", "add(1,2)", "variable_a". Statements are super-class of expressions. A statement is a line or a block of Birdee code, but not necessarily has a value. For example, "dim a as int", "if a>0 then ...". A type specifier represents a typename and usually appears after "as" key word in Birdee code.

To use the script as an expression, see the following example:

```vb
dim a as int = {@set_ast(expr("56"))@}
```

The script in "{@" and "@}" will generate an expression "56" as an integer and the script will be replaced by this expression when compiled. The functions "set\_ast" and "expr" are built-in functions defined in Birdee compiler's embeded Python interpreter. The Python function "expr" accepts a string and compiles the string as the source code of an expression in Birdee. It returns an AST node for Birdee compiler. (An AST node is an element of the parsed Birdee source code.) The function "set\_ast" takes an AST node as input and make Birdee compiler to insert the AST node at the position of current script in the Birdee source code. Note that in "{@" and "@}", any valid Python code is allowed. You can even use Python3's "input" function to get input from the user when the source code is compiled. You can also define functions in embedded scripts. Note that all Python code in "{@" and "@}" is executed in the same "top level" of Python interpreter. Defining variables in one script will affect all scripts executed later.

Next let's see a more complex example:

```vb
class student
	private id as int
	public function __init__(id as int)
		this.id=id
	end
end

{@
cnt=0
def counter():
	set_ast(expr(str(cnt)))
	cnt+=1
@}

dim stu1 = new student({@counter()@})
dim stu2 = new student({@counter()@})
dim stu3 = new student({@counter()@})
```

In this example, we have created a python function "counter", which generates an increasing integer after each call to it. Then we create 3 instances of "student" class and statically assign IDs to them. Then parameters for the constructor of  "student" class for stu1 to stu3 will be "1", "2" and "3". Note that in C++, it may introduce much more lines of code to implement such compile-time template-based counter.

Statements without values can also be compiled by Birdee's embedded scripts. Instead of using the function "expr", the function "stmt" can be utilized to compile a python string to an AST node. For example:

```vb
dim cmpstr= "hello"
{@
user_input=input()
set_ast(stmt(f''' if cmpstr == "{user_input}" then
	println("It is hello!")
end
'''))
@}
```

The above example can be compiled by Birdee with Python 3.6+. It gets an input from the user and generate an "if" statement block to compare with a Birdee string variable. 

A script in Birdee can be used as a type specifier. Similar to statement and expression generation, you can use the functions "set\_type" and "resolve\_type" in the scripts to use script as a type. "resolve\_type" accepts a string and will compile the string as a typename. It returns a "ResolvedType" object which is an internal representation of a type in Birdee. "set\_type" accepts an "ResolvedType" object and makes the compiler replace the current script with this type. Here is an example in which defines an integer variable using these two functions:

```vb
dim v as {@set_type(resolve_type("int"))@}
```

We can do some "meta-programming" with the scripts. The following example implements a compile-time function to get and use the return type of any function (the counterpart of C++ meta-programming "std::result\_of"):

```vb
{@
def result_of(str_expr):
	exp = expr(str_expr).get()
	ty = exp.resolved_type
	prototype = ty.get_detail()
	assert(isinstance(prototype, PrototypeAST)) # make sure the expression points to an function
	set_type(prototype.return_type)
@}

function func1() as int
	return 0
end

dim v as {@result_of("func1")@}  # should be int
```

As we can see in this example, we can access the type of an expression by the "resolved\_type" field and we can get the return type of an function prototype by its "return\_type". AST nodes are objects that can be accessed by the python script and details of the nodes can be found in the fields of them. We will later discuss the details of AST nodes.

The python scripts of Birdee can import existing Python modules by normal "import" command of python. Note that the directory "$BIRDEE_HOME/pylib" has been added to the python module search path of Python interpreter.

## 9.2 Scopes of the scripts

As we know, variable names in Python are scoped. For example, you cannot use a local variable defined in a Python function in the code outside of the function. Similarly, the embeded scripts of Birdee follows the scopes of the Birdee code.

 * The Python interpreter uses the "context" to manage the mapping from variable names to the actual data.
 * Once the Birdee code enters a new level of basic block (e.g. the compiler enters a function/the body of "for" loop) or scope (e.g. the class definition), the Python interpreter will copy the Python context (the "name" - "variable" map) of the previous context.
 * The scripts in the child context (in other words, in the lower level of the scopes) can have access to the names defined in parent Python contexts. But the scripts in the parent context cannot access the names in children context.
 * The scripts in the same level of Birdee scope/basic block shares the same Python context.

See the following example:

```
{@a=1@}
function f()
   {@print(a)@} # Okay, "a" is defined in the parent context
   print("Hi") # Birdee code
   {@b=1@} 
end
{@print(b)@} # Error here! "b" is defined in the children context
``` 

The modules of Birdee has different Python contexts, which means that you cannot directly use the Python names defined in scripts in other Birdee modules. But there is a way to do so and we will introduce it later in this chapter.

## 9.3 Annotations

Annotations can be applied on statements or expressions of Birdee programs. The syntax to use annotation is:

```vb
@annotation_name1
@annotation_name2
....
expression_or_statement
```

or 

```vb
@annotation_name expression_or_statement
```

In a word, annotations can be used by prepending `@annotation_name` to an expression or statement. The annotation and the annotated code can be separated by a new line or not. You can add multiple annotations to the same part of code (the multiple annotations will be called in the order in the source code). A parsed Birdee program will be translated in the form of AST (Abstract Syntax Tree) form. Each expression or statement is a node of the AST.

To process the annotation, in the compile time, the compiler will evaluate the `annotation_name` as a Python expression. The `annotation_name` can be a Python function name, a call to a Python function generator or any Python code that returns a callable Python object. The annotated part of code (in the AST node object form) will be applied to the Python function specified by the annotation name. The python function or callable object called by an annotation should take a parameter which is an AST node object. We show an example of an annotation that prints the type of the annotated AST node. You should first define or import a python function in an embedded script. Then call it with an annotation in Birdee code.

```vb
{@
def show_ast(node):
	print(type(node))
@}

@show_ast
function func1()
	println("hello")
end

@show_ast
dim var as int

@show_ast
func1()

{@
def show_ast2(prefix):
   return lambda node: print(prefix + type(node))
@}

@show_ast2("prefix")
function func2()
   println("hello")
end
```

After compiling the above code, the compiler will print "FunctionAST", "VariableSingleDefAST" and "CallExprAST" to its stdout. Note that the three annotations are executed in the same order as they are used in the Birdee source code. In the example, we first define a python function named "show\_ast" in the script. It has one argument which is the annotated AST node. Here we simply get and print the type of the AST node with python's built-in keyword "type" and "print". Then in the Birdee part of code, we apply the annotation "show\_ast" on a function definition, a variable definition and a function call expression. Finally, after compiling the code, we can see the type names of these AST nodes.

This annotation example only reads the AST nodes. Useful tools can be developed by using read-only annotations, like code safety checking. We can do more than that because annotations can modify the annotated AST! In the next example, we define and use an annotation which:

 * Can be applied on variable definitions
 * Checks if the variable is a string and is not initialized when defined
 * If so, add an initializer to the variable definition, and initialize the string variable to string "(null)"

```vb
{@
def add_initializer(node):
	if isinstance(node,VariableSingleDefAST):
		if node.value is None and node.resolved_type == resolve_type("string"): 
			#if node's initializer is null and its type is string
			node.value = expr('"(null)"'')
	elif isinstance(node,VariableMultiDefAST)): #VariableMultiDefAST is multiple variable definitions in one line
		for sub_def in node.lst: 
			#node.lst is the list of VariableSingleDefAST contained in VariableMultiDefAST
			add_initializer(sub_def)   #apply the VariableSingleDefAST to the current function 
@}

@add_initializer
dim a as string
println(a)

@add_initializer
dim b as string, c as string="hi"
println(b)
println(c)

```

We first define a python function. It accepts a single variable definition (VariableSingleDefAST) or multiple definitions (VariableMultiDefAST) in a line of code. Then for single variable definitions, the function check the initializer expression (node.value) and the variable type (node.resolved_type), and sets the initializer to string expression "(null)" if the variable has no initializer and the variable type is "string". For multiple definitions, the AST node (VariableMultiDefAST) contains a list of VariableSingleDefAST for each of the variable definitions. Our function iterates on the the list of contained VariableSingleDefAST to add the initializer. Once again, some fields of the AST node objects are used in this example. We will discuss about their details later. 

We apply the annotation on a single variable definition "a" and multiple definitions "b" and "c". Since variable "c" already has an initializer, the annotation has no effect on it. For variables "a" and "b", they will have an initial value "(null)" after the annotation has been applied.  

If you apply an annotation on a template, the corresponding Python function will not be called with the template AST, but when each of the instance of the template is constructed, the Python function will be called on the instance AST.

Please be careful on how Birdee compiler split the annotation from the annotated Birdee code. The annotation starts from "@" and ends at the first encountered space (" ") or newline. So if your annotation contains a space, the code will not work as expected. For example, the following code will not compile, because there is a space in "The prefix":

```
{@
def show_ast2(prefix):
   return lambda node: print(prefix + type(node))
@}

@show_ast2("The prefix")
function func2()
   println("hello")
```

## 9.4 Init scripts and importing as modules

By default, Birdee will dispose all embeded Python scripts which are not in templates. So the following module will compile, but will not function:

```
{@
def somefunction(ast):
   pass
@}

@somefunction
func hello[T](h as T)

end
```

If we compile the above code in a Birdee module "hello" and we import "hello" in another module, a Python error will be thrown, saying that the name "somefunction" is not found. The cause is that the script in the top level that defines the function is lost. To preserve the script, you should add an annotation `init_script` on the embeded scripts. Note that `init_script` is a special annotation defined by Birdee compiler, so you cannot create a user-defined annotation with the same name. `init_script` can only be applied on top-level embeded scripts. Otherwise, the compiler will fail on the program.

You can mark multiple top-level scripts as `init_script`, the scripts will be executed after it is imported by other modules. The multiple scriptts will be executed in the original order defined in the source code. The above code can be modified to work:

```
@init_script
{@
def somefunction(ast):
   pass
@}
...
```

Remind that the different modules in Birdee have independent Python contexts. Even after importing a module, the init scripts will not affect the Python context in the current module. So in the above example, the function "somefunction" can only be used in the scripts in the module "hello". However, you can import the Python names defined in other modules' init scripts by importing them:

```
import hello

{@from hello import somefunction@}

@somefunction
function a()
end
```

In the above example, the init scipts of the module `hello` is imported as a Python module by the line `{@from hello import somefunction@}`. The module names of the Python modules can be derived from the names of Birdee modules where the scripts are defined. You just need to replace the "." in the Birdee module names with "\_0". For example, for Birdee module "org.birdee.a", its corresponding Python module is `org_0birdee_0a`. 

## 9.5 Generative script

It is an interesting feature that the Birdee compiler itself can be used as an python library. Python programs can use Birdee to generate Birdee programs. We call these python programs "generative scripts". There are two ways to use Birdee in python generative scripts. The first way is to use Birdee compiler in Python interpreter mode. You need to add a switch "-s" to the command line parameter of Birdee compiler to activate this mode. When "-s" switch is given, the compiler will treat the input file as an Python program and run the input script with Python interpreter. The second way to run generative scripts is to simply run the standard python interpreter and import the module "birdeec" in the python code. Note that the Birdee binary library for python binding must be available in Python's library search path. And the library file must be named "birdeec.so" (on linux) or "birdeec.pyd" (on Windows). All APIs used in embedded scripts that are discussed above can also be used in generative scripts. Birdee compiler exposes some APIs that are only available for generative scripts. They are designed for compiler control. Here is an simple example of a generative script.

```python
from birdeec import *
top_level('''
dim str = "hello world"
println(str)
''')
process_top_level()
generate()
clear_compile_unit()
```

To use Birdee as a python library, you must first import the module "birdeec". (Note that for embedded scripts, this module has been automatically imported.) You can compile the top level code by function "top\_level". It accepts a python string of Birdee source code. The function returns nothing, but the top-level code will be stored in the compiler library after "top\_level" is called. Then, you should use function "process\_top\_level" to build the AST of the program and check the potential errors in the source code. The next step is to call function "generate" to generate the target object file. Optionally, if you want to re-use the current python process to compile another Birdee program, you should call the function "clear\_compile\_unit" to clear the internal states of the Birdee compiler library. This function is useful when you need to compile multiple Birdee programs in one script for testing. 

Our Birdee compiler project has been using generative scripts to test the features and expected behaviors of the compiler.

## 9.4 Error handling

We have already introduced several APIs for python to compile a string of Birdee code to Birdee programs, including "expr", "stmt", "top\_level", "process\_top\_level". An erroneous Birdee program string will fail to compile and an Python exception will be raised when calling these functions. You can catch the exception in both embedded and generative scripts using python's exception handling mechanism. The basic example to catch compilation errors is shown as follows:

```python
try:
	...
	may_throw(...)
	...
except TokenizerException:
	e=get_tokenizer_error()
	print(e.linenumber,e.pos,e.msg)
except CompileException:
	e=get_compile_error()
	print(e.linenumber,e.pos,e.msg)
```

Catching a TokenizerException means that the compiler meets an unrecognized token in the program. An CompileException means that the program is somewhat invalid.

The function "may\_throw" is any function that may raise an compiler exception, which include:

 * expr - may throw TokenizerException and CompileException
 * stmt - may throw TokenizerException and CompileException
 * top\_level - may throw TokenizerException
 * process\_top\_level - may throw CompileException

Note that the exception object in the "except" clause in python for Birdee does not contain detailed exception informantion. You can either use function "get\_tokenizer\_error" or "get\_compile\_error" to get detailed exception information like the position of the error and the explanation of the error.

Important note: Once the compile raises an exception, the compiler is in an unrecoverable state. You must not continue to use the compiler instance and you must call "clear\_compile\_unit" to reset the state of the compiler before continue to use "birdeec" library.

## 9.5 Memory management and pointer ownership

There are two memory management systems in Birdee's compiler. One is the core part of the compiler which is written in C++. It uses unique\_ptr to manage the lifetime and the ownership of the AST nodes. The other is the python interpreter's memory management system which uses reference counting. It is challenging to combine these two memory systems. For example, after we allocate an AST in python (e.g. by "expr" function), how can we move the ownership of the AST object to the C++ unique\_ptr? We address this problem by introducing "pointer ownership" in python binding of Birdee. Having ownership of an object means that the respective memory management system is responsible to free the space for the object. If a parent AST node has the ownership pointer to another children AST node, it means that when the parent node is destroyed, the children nodes of it must be destroyed by it. If a python variable has the ownership pointer to an AST node, you can transfer the ownership of the pointer by assigning the ownership pointer to an existing AST node's ownership pointer field. Or you can use some APIs like "set\_ast" to transfer the ownership to the compiler.

For most of the fields in AST nodes, getting the fields in python gives you a reference. In this section, we refer to the term "reference" as the pointer to an AST node without ownership. Note that reading an ownership pointer field of an AST node in python returns a reference to the object (not the ownership pointer!). For example, a Birdee variable definition is represented by the AST node class "VariableSingleDefAST" and the initializer of the variable can be accessed by the field "value" as an ownership pointer. You can get the reference to the initializer expression by:

```python
var_ast = some_variable_def_ast
init_of_var = var_ast.value
```

The functions like "expr" and "stmt" returns an ownership pointer to the compiled AST. You cannot directly access the fields of the AST node by an ownership pointer. However, you can use the "get" method of an ownership pointer in python to get the reference to the AST object and then access the fields with the reference.

```python
exp=expr("123")
print(type(exp)) #should be "StatementAST_UniquePtr"
print(type(exp.get())) #should be "NumberExprAST"
```

Writing to an ownership pointer field of an AST node requires an ownership pointer in python. You cannot write to an ownership pointer field with a reference. After ownership transfer from a python ownership pointer, the python ownership pointer will point to null value. Take the class "VariableSingleDefAST" as an example:

```python
var_ast = some_variable_def_ast # it has type VariableSingleDefAST
exp=expr("123") #an ownership pointer to an expression
var_ast.value = exp.get() #fails to run here! It requires an ownership pointer, not a reference
var_ast.value = exp   #okay, ownership transfer from exp
assert(exp.get() is None) #exp is empty after transfering ownership
```

## 9.6 Detailed API specification

TBD