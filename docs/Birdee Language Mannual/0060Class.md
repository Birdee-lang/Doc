# 6. Class

## 6.1 Basic definitions

A class is a structured collection of data. Related variables can be include in a class. To define a class, use the below syntax:

```vb
class {class_name}
	...
end [class]
```

The body of a class ("..." in above syntax) can be declaration of member variables and member functions.
To define a member variable, use the below syntax within the "class ... end":

```vb
public {variable_name} as {type}
```

or 

```vb
private {variable_name} as {type}
```

Note: current version of Birdee does not allow assign initial values to member variables. 

The "public" and "private" keywords specify the access modifier of the variable. We will discusses it a bit later. Now we write our first class:

```vb
class bird
	public name as string
	public weight as float
end
```

The class describes a bird with a name and a weight. You cannot use the member variables until you create an instance of the class. To create an instance of "bird" class, use "new" keyword:

```vb
dim mybird as bird = new bird
mybird.name = "Birdee"
mybird.wright = 2.3
println(mybird.name)
```

The member variables can be accessed by a "." after a class instance expression, with the name of the member variable.
The above example creates an instance of class "bird" and assign it to a variable "mybird" (Note that the class name can be used as a type!). The example then assign the member variables of "mybird". The member variables belongs to the instances of the class. Thus, you cannot use them without a reference to an instance.

## 6.2 Explanations of terminologies

### 6.2.1 Class v.s. Class Instance v.s. Object

As defined above, a class is the definition of structured data. A class (for example, "bird" class) is a type for some data.

Class instance is a piece of concrete data of some class. "string" is a class, and a string variable is a class instance of string. Class instances are sometimes called "Objects".

Note that different instances of a class are independent with each other. Assigning the member variable of one instance will not affect the same member variable of others. 

### 6.2.2 Member variables & fields

They have the same meaning.

### 6.2.3 Member functions & methods

They have the same meaning.

### 6.2.4 Reference

Once an object is allocated in memory, how can you find it and operate on it? The answer is by "reference". A reference points to a instance of class (or an array). Variables of class/array types holds the references to the instances in the memory (e.g. dim mybird as bird). So copying variables of class/array types copies the references to the instances, instead of copying the object/array themselves.

Here we introduce a special reference constant "null". It points to actually nothing. You can assign null to any reference typed variable (class, array). If a reference is null, it means that it is an empty value. You should never call a member method or get the field of a null reference, or an error will occur. You can check if a reference is null by:

```vb
if some_reference !== null then
...
end
```

Note that for class object variables, the initial values are null.

### 6.2.5 "this"

"this" is a reference to the current object in the member function. It is a Birdee keyword that can only be used in member functions. For Birdee code

```vb
obj.funct()
```

when it is run, in the method "funct", "this" will point to the object being called - in this case, "obj".

## 6.3 Member functions

Member functions can be defined within the scope of the class:

```vb
{access_modifier} function {function_name} ([ {parameter1} as {type}, {parameter2} as {type}, ... ]) [as  {type}]
	...
end [function]
```

It is similar to the definition of normal functions, except that member functions are defined inside a class, and there is an "access_modifier" at the beginning of the function. The "access_modifier" can be "public", "private" or omitted (which will be explained later). 

We can write a member function in the above "bird" class:

```vb
class bird
	public name as string
	public weight as float
	public function fly()
		println("my name is "+ this.name + ". I am flying!")
	end
end
```

You can call "fly" on an instance of "bird" class:

```vb
dim mybird as bird = new bird
mybird.name = "Birdee"
mybird.wright = 2.3
mybird.fly()
```

The above example of "bird" class uses a keyword "this" of Birdee to represent the current instance calling the member function (see the line: println("my name is "+ this.name + ". I am flying!") ) . When the member function is called ( mybird.fly() ), "this" will be a reference to the instance pointed by "mybird". "this" keyword cannot be used outside the class definition. Also, in the member functions, to use the member variables, "this" keyword can be omitted. For example "this.name" can be simplified to "name", when name is a member variable of the same class. Also note that for a member function, if there is a local variable (variables defined in the body of a function or in the arguments) having the same name of a member variable, using the name will result in using the local variable. To use the member variable have a conflicting name, use "this.XXX" instead. (XXX is the conflicting name).

Note that the member functions cannot be accessed without an instance.

## 6.4 Access modifier

We finally explain what is "private" and "public" in front of the member variables and functions. If a member variable or function is defined "private", no one can access it unless it is a member function of the same class, for example:


```vb
class bird
	public name as string
	public weight as float
	private secret_name as string
end

dim mybird as bird = new bird
mybird.secret_name="sadas"
```

The above code will not compile, because "secret_name" is a private member. Access control (making members private/public) is useful when you have some internal variables or functions that you do not want other people to have access to. But remind that a member function can always access all members of the same class, regardless of being private or public.

Note that a member function's access modifier can be omitted when defining it, and the access is set to "public" by default.

## 6.5 Initialization & destruction

We can now create instances by "new". But what if we want to initialze the object while creating it? The syntax of "New with initialization" can be use.

```vb
new {class_name}.{method_name}([arg1,arg2,....])
```

The class_name is the name of the class of the instance to be created. The method_name is the member function name to be called after the creation of the object. The function should be a public function. Here is an example:

```vb
class bird
	public name as string
	public weight as float
	public void init(name as string, weight as float)
		this.name=name
		this.weight=weight
	end
end

dim mybird as bird = new bird.init("birdee",2.13)
```

The above example creates and initializes a "bird" object, which is equivalent to:

```vb
dim mybird as bird = new bird
mybird.init("birdee",2.13)
```

Note: the "New with initialization" will discard the return value of the called function and will always return the created instance.

When the class has a "\_\_init\_\_" method defined in the class body, users can use "New with initialization" in a simpler way. The following code:

```vb
dim obj = new SomeClass("hi")
```

will create an new instance of class SomeClass and call obj.\_\_init\_\_("hi") for initialization. 

If the "\_\_init\_\_" method has no arguments, users can further save the "(...)" for calling "\_\_init\_\_":

```vb
class SomeClass
    public function __init__()
    end
end
dim obj = new SomeClass   #__init__ will be called here.
```

Note that "\_\_init\_\_" must be a public function in the class.

Birdee uses garbage collection for memory management, which means when the object created is no longer used (when there are no references pointing to the object), the object will be automatically delected. If the object has "\_\_del\_\_" method defined, when it is deleted and, the method will be called. You can do some finalization work in this method.

## 6.6 Operator overloading

Operator overloading means you can apply basic operators (like +,-,\*,/,...) on you own class and define your classes' own behavior on them.

We are acutally familar with the use of operator overload, which is used in "+" operator of string. (Don't forget that "string" is simply a class defined by Birdee!). We can use "+" to concatenate two strings:

```vb
println("string A" + "string B")
```

There is no magic in it and Birdee implements it using operator overload.

To overload operator "+", you must define a special member function "\_\_add\_\_" in your class:

```vb
class complex
	public real as double
	public imaginary as double

	public function __add__(complex other) as complex
		return new complex:set(this.real + other.real, this.imaginary + other.imaginary)
	end

	public function set(real as double,imaginary as double)
		this.real=real
		this.imaginary=imaginary
	end
end

dim v1 = new complex:set(1,3), v2 = new complex:set(2,4)
dim v3 = v1 + v2
println(double2str(v3.real))
```

The above example defines a complex number class. It overloads "+" by the function "\_\_add\_\_". The function accepts another "complex" object and creates a new complex object as the result. It then creates two complex numbers (1,3) and (2,4), add them and store in variable "v3" and print the real part of the result. The line "dim v3 = v1 + v2" will be equivalent to:

```vb
dim v3 = v1.__add__(v2)
```

There are other operators that can be overloaded. The operators and functions to implement are listed in the below table:

|  Operator |  Function Name to implement | Operand |
|---|---| --- |
|+|\_\_add\_\_|The other object|
|-|\_\_sub\_\_|The other object|
|*|\_\_mul\_\_|The other object|
|/|\_\_div\_\_|The other object|
|%|\_\_mod\_\_|The other object|
|==|\_\_eq\_\_|The other object|
|!=|\_\_ne\_\_|The other object|
|>=|\_\_ge\_\_|The other object|
|<=|\_\_le\_\_|The other object|
|>|\_\_gt\_\_|The other object|
|<|\_\_lt\_\_|The other object|
|\|\||\_\_logic_or\_\_|The other object|
|\||\_\_or\_\_|The other object|
|&&|\_\_logic_and\_\_|The other object|
|&|\_\_and\_\_|The other object|
|^|\_\_xor\_\_|The other object|
|!|\_\_not\_\_|None|
|Array read|\_\_getitem\_\_|The "index"|
|Array write|\_\_setitem\_\_|The "index" and the object to "put" in|

For a class object, if an operator is applied, the corresponding method will be called. If the method is not defined or it is private, a compile error will be raised.

For overloaded binary opeartors (operators with two opearnds), "A ? B", where "?" is any binary operators, is equivalent to "A.\_\_XXXX\_\_(B)", where "\_\_XXXX\_\_" is the corresponding method. For overloaded unary operators (operators with one operand), using them is equivalent to "A.\_\_XXXX\_\_()", where "\_\_XXXX\_\_" is the corresponding method.

Operators for array read & write are special cases for operator overloading. These two operators overloads the "\[\]" operator which is originally used for accessing array elements. If the indexed object is to be read "from" an object, the "\_\_getitem\_\_" method will be called. For Birdee code like

```vb
dim a = obj["123"]
```

where the variable "obj" is not an array, the compiler will first the "\_\_getitem\_\_" method of the class of "obj". The actual generated code will be 

```vb
dim a = obj.__getitem__("123")
```

Similarly, if the indexed element is written (on the left of the "="), the method "\_\_setitem\_\_" will be called. The first parameter should be the index and the second parameter should be the value to be written to the object. For example, the following two lines of code have the same effect:

```vb
obj["123"]=34
obj.__setitem__("123",34)
```

The existence of the method "\_\_setitem\_\_" is optional, as long as you never "write" to an indexed element. However, if you want to overload "[]", the method "\_\_getitem\_\_" should always be defined in the class.


Note that the type of parameters of methods for operator overloading is not necessarily the same class of the current class. They can be any valid types.

Also note that you can use function templates for operator overloading. Templates will be later introduced.

## 6.7 Class inherit
### 6.7.1 Basic inherit
Class can inherit another class's public members through class inherit. The inherited class is also called parent class. A class with parent class is usually viewed as a specification of its parent. 

The following codes:

```vb
class ParentClass
    private a as int
    public b as int
    public function get() as int
        return a
    end
end
class SomeClass : ParentClass
    public function __init__()
    end
    public function get2() as int
        return b + get()
    end
end
```

define a class named "SomeClass" with a parent class named "ParentClass", and SomeClass inherits member field "b" and member function "get()" from ParentClass, note that member field "a" is not inherited since it's private to ParentClass. Also, SomeClass can access the member it inherits inside class directly, as showed in the example.

Besides, the following code:

```vb
class ParentClass
    public function __init__()
    end
    public function __del__()
    end
    public function __not__() as boolean
        return true
    end
end
class SomeClass : ParentClass
    public function __init__()
    end
    public function __del__()
    end
end

dim foo = new SomeClass
dim bar = !foo
```

shows an example of class inherit with special member functions. Note that in code "dim foo = new SomeClass", the function __init__() of SomeClass will be automatically called, but __init__() of ParentClass will not! Also, the __del__() of SomeClass will be automatically called when garbadge collected, but __del__() of ParentClass will not. And the code "dim bar = !foo" won't compile because SomeClass does not contain an operator overloading function for !. Even if SomeClass inherits one from ParentClass, the compiler will not automatically call it.

### 6.7.2 "super"
What if we want the functions in parent be called during above scenarios? We can use the "super" keyword. Similar to "this" keyword, "super" keyword represents a built-in reference inside class. However, "this" refers to the instance itself, while "super" refers to the parent part of the instance. That is to say, we can use "super" keyword to only access the members inherited from parent. 

With "super" keyword, the above code can be modified to:

```vb
class ParentClass
    public function __init__()
    end
    public function __del__()
    end
    public function __not__() as boolean
        return true
    end
end
class SomeClass : ParentClass
    public function __init__()
        super.__init__()
    end
    public function __del__()
        super.__del__()
    end
    public function __not__() as boolean
        return super.__not__()
    end
end


dim foo = new SomeClass
dim bar = !foo
```

Then, the code will compile and the special functions of parent will be called automatically.

## 6.8 Run Time Type Information (RTTI)

You can get the type information of an object at the run time, as long as the type of the object has Run Time Type Information (RTTI) enabled. The RTTI describes the name and the inherience information of a class. By default, RTTI will not be generated for classes unless the classes has virtual functions. You can manually enable RTTI on a class by adding "@enable_rtti" before the "class" keyword of a class. The following example shows three classes with RTTI. Note that classes with virtual functions automatically include RTTI. 

```vb
@enable_rtti
class A
end

class B
   @virtual public function b()
   end
end

class C
   public c as int
   @virtual public function b()
   end
end
```


Note that if a class has RTTI enabled, all classes extending (inheriting from) it will be automatically marked RTTI-enabled. If a class is manually marked `enable_rtti`, either it has no parent class, or it should extend a class with RTTI.

Given an expression, the RTTI data can be fetched by the keyword `typeof`. The returned value of `typeof(some_expression)` is an object of class `type_info`, which contains the RTTI of the class of the expression. The class `type_info` has a method to get the name of the class - `get_name`, and it has a method

`public function is_parent_of(child as type_info) as boolean`

to check if another class (represented by RTTI) is inherited from the current class. Also, the `type_info` class has a method 

`public function get_parent() as type_info `

to get RTTI of the parent class. If a class has no parent class, the method returns null. See the following example:

```vb
dim a as B = new C
println(typeof(a).get_name()) # should print "XXXX.C"
println(typeof(a).get_parent().get_name()) # should print "XXXX.B"
```

The variable "a" is declared as an object in class B. But it is assigned with an instance of class C. Using `typeof` operator, we can get the exact type of the variable "a".

The `typeof` operator will execute the expression and extract the reference to the RTTI object at the run time. The expressions to be evaluated by `typeof` should be of classes with RTTI, otherwise the compiler will throw an error.

A unique RTTI object will be created for each different class. Class template instances are different classes with different RTTIs.  

Given a type, the RTTI can be fetched by a special function `get_type_info[T]` defined in module `rtti`. You can import this function by `import rtti:get_type_info`. You need to replace `T` with the class you need to fetch for RTTI. `T` can only be classes with RTTI.

RTTI is useful when a variable is assigned with a subclass of the class which the variable is defined. Developers may want to check if the variable really holds an object in a subclass. Since RTTI for a class is unique, we can compare the references of `type_info` (RTTI) by `===` to check that:

```vb
import rtti:get_type_info
dim a as B = new C
if typeof(a)===get_type_info[C]() then
   println("the variable a is of class C")
end
```

The subclass checking and safe down-casting can be done with RTTI. Birdee provides the function `dyn_cast` in the module `rtti` to safely convert a superclass reference to a subclass reference.

```vb
import rtti:dyn_cast
dim a as B = new C
dim c as C = dyn_cast[C](a)
priintln(int2str(c.c))
```

The above code converts a variable "a" of superclass "B" to variable "c" of subclass "C", using `dyn_cast[C]`. The function `dyn_cast[C]` will convert the reference in the parameter to a reference of class "C". If the object pointered by the given parameter is not an instance of "C" or subclass of "C", `dyn_cast[C]` will return null. You can replace 'C' here with other classes with RTTI. `dyn_cast[...]` is a system provided function, which internally compares the RTTIs of the classes.

Enabling RTTI has some overhead in space. If a class has RTTI, all of its instance has one additional hidden member pointing to the `type_info` object of the class.

## 6.9 Abstract Class & Interface

TODO

## 6.10 Structs

Struct is a similar but different concept as class. Structs can be similarly defined as classes. You just need to replace "class" keyword with "struct".

```vb
struct {name}
...
end [struct]
```

The member variables and functions can be similarly defined and used in structs. 

So what's the difference between struct and class? One key difference is that for local variables defined in functions, structs are allocated on the stack and class objects are allocated on the heap. The access and allocation of data on the stack is much faster than on the heap. Also, once the program leaves the scope of a function, the space of the local struct variables will be deallocated. 

The second difference is that, in the context of Birdee, variables of "class" has "reference semantic", while variables of "struct" has "value semantic". A class variable (including local, global and member one) is always a reference to an object in the heap or null. If you copy a class object variable, you just copy the reference to the object, not the actual data of object. On the other hand, copying struct variables (by operator "=") or implicitly copying struct variables (in function parameters), you will copy the whole struct object. Hence, struct variables are "values", not "references".

If a class/struct, say "A", has a class, say "B", member variable, the class/struct A only holds a refernce to B. But if "B" is changed to struct, "B" will embeded into the memory layout of "A", which means allocating an object of "A" will implicitly allocate space for "B". 

So you should be careful when the struct has many fields - copying these structs involves large amounts of memory copying.

Some notes on struct:

 * Operator overloading is supported in structs.
 * You cannot enable RTTI on struct

Important: The "\_\_del\_\_" methods of structs will not be automatically called when the structs objects are destroyed!