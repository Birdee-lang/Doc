# 8. Templates

We explain the use of templates with the following example. Assume we have a function doing addition on int:

```vb
function add(a as int, b as int) as int
	return a+b
end
```

What if we want a function adding floats? We may re-write an similar function:

```vb
function add(a as float, b as float) as float
	return a+b
end
```

What if we want a function adding strings, or custom class objects? Writing so many functions that is almost the same? We can use templates!

## 8.1 Function Templates

Templates can define more general version of functions. To define a function template, users should use the syntax:

```vb
function {function_name}[{placeholder1},{placeholder2},...]({arguments}) as {return type}
	...
end
```

In the context of Birdee templates, the "placeholders" refers to the template parameters. The templates allow types to be the parameters of functions and classes. The difference between normal functions and function templates is that function templates has "[...]" part for the template arguments. Here is an example for adding two paramters:

```vb
function add[T](a as T, b as T) as T
	return a+b
end
```

In the function header, we declare a template parameter for typename by "T", and we can use the template parameter as a type in the rest part of the function. To use the template function, add "[...]" to the function name to give the parameters to the template parameters. For example:

```vb
add[int](1 , 3)
add[string]("haha","yoyo")
add[float](1.2 , 3.4)
```

In addition, the template parameter in the function template can be actual constant values rather than typenames. As is shown above, to declare a typename template parameter, users need to insert an identifier in the "[...]" of the function header. Similarly, declare a constant value template parameter, insert "{identifier} as {type}" into "[...]". Note that Birdee only support basic types (integers/floats) and string as the type of constant value template parameter. For example, we modifier the template above a little:

```vb
function add[T,offset as int](a as T, b as T) as T
	return a+b+offset
end
```

We add a constant value template parameter "offset", and use it in the returned expression. Then we use it with:

```vb
add[int,3](1 , 3)
add[float,66](1.2 , 3.4)
```

However, the code "add\[string,123\]("foo","bar")" will not compile, since adding an integer to a string is not allowed. Also note that the arguments to the constant value placeholders can only be literal numbers/strings, such as "12.34" or 12.34.

Member functions of a class can also be a template:

```vb
class myclass
	private value as int
	public function add[T](v as T)
		value = value + v
	end
end
```

## 8.2 Class Templates

Similarly, one can define a class template with the following syntax:

```vb
class {class_name}[{placeholder1},{placeholder2},...]
	...
end
```

The placeholders can be similarly used in the scope of the class. Here is an example for a class for linked list:

```vb
class list[T]
	public head as list_node[T]
	public tail as list_node[T]
	public function push_back(v as T)
		dim node=new list_node[T]
		node.v=v
		node.next=null
		if head===null then
			head=node
			tail=node
		else
			tail.next=node
			tail=node
		end
	end
	public function create()
		head=null
		tail=null
	end
end

class list_node[T]
	public v as T
	public next as list_node[T]
end
```

You can use it with:

```vb
dim mylist = new list[string]:create()
mylist.push_back("hi")
println(mylist.tail.v)
```

Note that "list\[string\]" and "list\[int\]" are different types and not related.

## 8.3 Template type parameter deduction

In some cases, you can call an template function without giving some of the template arguments. Birdee compiler can infer the template arguments from the function arguments. For example, for function:

```vb
function add[T](a as T, B as T) as T
	return a+b
end
```

We can call the template function with:

```vb
add(1,2)
```

Since the type of arguments of the function has been given ("1" and "2" are of type "int") and in the template they are declared as "T", Birdee can infer that the template parameter "T" for the template "add\[T\]" is "int". So the code above has the same effect as:

```vb
add[int](1,2)
```

Template type parameter deduction has some restrictions.
 * It only applies to function templates. 
 * The template parameters to be automatically inferred must be used in the function's parameters. The template parameters can be infered even it is in a complex type in the function parameter. For example, Birdee can infer the template parameter "T" in the following code:
```vb
function get[T](a as T[]) as T
	return a[0]
end
```

 The function parameter "a" is declared as a complicated type "T\[\]". For the given function parameter `int[]`, the compiler can infer T as int. Besides the array type, the template parameter deduction on closure types and functypes are supported.
 * You can manually give some of the template arguments and leave other template arguments to be inferred by Birdee compiler. However, the template parameters to be automatically inferred must be the last several template parameters declared in the template's parameter list. For example, if we need a function template to add two values and convert the value to a specific type, the code can be:

```vb
function add[RetT,T1,T2](a as T1, B as T2) as RetT
	return a+b
end
```

 We can use it by giving the argument for "RetT" and leave "T1" and "T2" inferred by the compiler. For example:

```vb
 dim v = add[double](123,3.45f)
```

 Make sure that the template parameters that are designed to be given by the caller manually must be the first few parameters in the parameter list.

 * A template type parameters cannot be inferred as more than one different types. For example, the following template will not compile:

```vb
function add[T](a as T, B as T) as T
	return a+b
end
add(1, 2.34) 
```

 Because "T" is first inferred as "int" then "float".
 * All template parameters should either be manually specified by the caller (with \[...\]) or inferred with function's parameters.

## 8.4 Variadic template and function

### 8.4.1 Variadic template

Sometimes it is useful to let templates have variable number of template parameters. For example, Birdee's standard library provides the "tuple" template struct which can combine value of multiple types in one variable. Here is an example for using "tuple":

```vb
import tuple:tuple
dim a as tuple[int,float,string]
a.v0 = 1 #v0 is int
a.v1 = 3.14 #v1 is float
a.v2 = "hello" #v2 is string

dim b as tuple[float,int]
b.v0 = a.v1 #v0 is int
b.v1 = a.v0
```

The "tuple" template can accept any number of argument types. In the example, we use two instances of tuple, with arguments "int,float,string" and "float,int".

The definition of the tuple template is something like:

```vb
struct tuple[...]
	#detailed definition omitted
end
```

Here the ellipsis "..." in the template parameter list lets the compiler know that the template is a variadic template. And the "..." accepts any number of template arguments.

You can still declare template parameters even if there is the ellipsis ("...") in the parameter list. However, the ellipsis ("...") should appear at the end of the parameter list.

```vb
# an example of declaring template parameters with "..."
struct vararg_struct[T1,T2,...]

end

# the following code won't compile
struct vararg_struct_bad[...,T1,T2]

end
```

You can use the compile-time scripts to get the argument types in "..." within a template instance, by Python code:

```python
targs = get_cur_class().template_instance_args
for targ in targs:
	...
```

Compile-time scripts will be discussed soon later.

### 8.4.2 Variadic function

The ellipsis "..." can also be used in function parameter list. Moreover, you can use "..." in function's parameter to let the function accept variable number of parameters. For example, a function template that returns the sum of all arguments can be:

```vb
func addn[...](...) as int
	return {@
arr_args=[]
for arg in get_cur_func().proto.args:
	arr_args.append(arg.name)
set_ast(expr(" + ".join(arr_args)))
@}
end
```

The code between "{@" and "@}" is the compile time Python script. The script expands "..." and returns an expression that adds the parameters all. We use function to add any number of values:

```vb
addn(1,2,3,4)
addn(1,2)
```

How does variadic function works? It is based on template type parameter deduction. When you call the function "addn" with "addn(1,2,3,4)", the "..." in the function parameter will be assigned with types "int,int,int,int". Then the ellipsis in the template parameter will be matched with the ellipsis in the function parameter. So the actual function to be generated in this example will be "addn[int,int,int,int]".

Another way to use variadic function is to use it in a member function of variadic template class/struct. For example, you can define a member function in "tuple" to set all fields in a tuple object:

```vb
struct tuple[...]
	#detailed definition omitted
	function set(...) 
		#detailed code omitted
	end
end
```

For a tuple type "tuple\[int,float\]", the member function "set" will be expanded as

```vb
	function set(___vararg0 as int, ___vararg1 as float) 
		#detailed code omitted
	end
```

Some rules for variadic functions should be followed:
 * Birdee creates an local varaible for each of the arguments of "..." in the function parameter, with names "\_\_\_vararg0", "\_\_\_vararg1", "\_\_\_vararg2", ... (starting with three underscores "\_"). 
 * Only template functions with variadic template parameters can have variadic function parameters.
 * The ellipsis should only appear in the end of function's parameter list.

### 8.4.3 Named variadic parameter

It is possible to create a variadic member function in a variadic class template. In the last section, we have already shown an example of using "..." in non-template function parameter to expand the variadic template parameters of the parent class. It is also allowed to create a variadic template function in a variadic class template. For example,

```vb
class cls[...]
	public function f[...](...)
	end
end
```

In this example, which ellipsis will be expanded in the ellipsis in the function parameter? In this case, Birdee will expand the template argument of function "f" to the parameter list. That is, "new cls[int].f[string]" will have a parameter type of "string". However, what if we want to expand the "..." in the function parameter using the class template parameters? Thus, we introduce named variadic parameter. An identifier can follow the "..." in template parameter lists and function parameter lists to specify which ellipsis the function parameter will be expanded as.

```vb
class cls[...CArg]
	public function f[...FArg](...CArg)
	end
	public function f2[...FArg](...FArg)
	end
end
dim obj = new cls[boolean]
obj.f[string](true)
obj.f2[string]("hello")
```

If a named function parameter ellipsis is expanded, it will first find a matched name in the current function template's named ellipsis. If no match, it will search for its name in 
the current class/struct's named ellipsis. If still no match, an compile error occurs.

If a named function parameter ellipsis is expanded, it will first try to match current function template's ellipsis. If current function is not a template instance, it will search match current class/struct's ellipsis.

## 8.5 Notes on Templates

Once users "call" a template with arguments, the compiler will instantiate an instance of the template. The template instance will be created only once for the same arguments to the placeholders in the same module. An instance of template will create an independent segment of code and data structures that is not shared by other instances.

The whole source code for the templates will be copied into the metadata file of the module after compiled, for the use of other modules.
