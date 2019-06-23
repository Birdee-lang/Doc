# Birdee playground

Birdee provides a "playground" environment to let you try Birdee code and instantly view the results of each line in your input. The playground is also called REPL (Read-Eval-Print-Loop), this explains what the "playground" does: for each line the user input, it executes the line of code and then print the result. Unlike the Birdee compiler, it will not produce an compiled program in a file. Instead, it runs the inputed code immediately.

## Opening playground

The playground is an console application which you can find in `$BIRDEE_HOME\bin\birdeeplay` (Linux) or `%BIRDEE_HOME%\bin\BirdeePlayground.exe` (Windows CMD). Run this program, then you should see the following in the console:

```
Birdee Playground
    ____  _          __
   / __ )(_)________/ /__  ___
  / __  / / ___/ __  / _ \/ _ \
 / /_/ / / /  / /_/ /  __/  __/
/_____/_/_/   \__,_/\___/\___/

You can try and see Birdee codes here. Type ":q" to exit.
birdee>>
```

If you cannot open Birdee playground, please check:

 * If the "BirdeeBinding.dll", "BirdeeCompilerCore.dll" (libBirdeeBinding.so and libBirdeeCompilerCore.so) is in the same directory of the playground executable file.
 * If Python with the required version is installed. For Windows, you need to add the root directory to the "PATH" environment variable.


## Try Birdee code

Now you can type in any valid Birdee code. After typing "Enter" key, the playground will compile and execute the line. For example, you can define a variable:

```
birdee>> dim a = 1
```

Then you can use the variable in an expression. If the code you entered is an expression, the playground will print the type and the value of the result:

```
birdee>> (a+1)*a
int: 2
```

You can define and use classes/structs and functions.

A class:

```
birdee>> class clsa
........ public v as string
........ public func __init__(v as string) => this.v=v
........ public func printit()=>println(v)
........ end
birdee>> dim b = new clsa("hi")
birdee>> b.printit()
hi
birdee>>
```

A function:

```
birdee>> function addint(a as int, b as int) as int
........ return a+b
........ end
functype (int, int) as int: Function addint
birdee>> addint(3,4)
int: 7
```

Even a function template: 

```
birdee>> function add[T](a as T, b as T) as T => a+b
(Error type): Function add
birdee>> add(1,2)
int: 3
birdee>> add("1","2")
birdee.string: 12
```

You can also import the compiled Birdee modules in the `BIRDEE_HOME`. Moveover, declaring existing C/C++ functions is allowed:

```
birdee>> declare function malloc(sz as uint) as pointer
functype (uint) as pointer: Function malloc
birdee>> import typedptr:*
........ dim p = mkref[int](malloc(4))
birdee>> p.set(32)
birdee>> p.get()
int: 32
```

The above code declares a C function - "malloc" in the standard C library, which allocates a piece of memory. Then, we import a Birdee module "typedptr" and set and get the allocated memory with the "typedptr" interface.

## Special commands

There are several special commands in Birdee playground designed to control the playground, which is not valid Birdee code. The commands include:

 * :q - Exit the playground
 * :printir=... - "..." can be true or false. This command controls whether playground should print the LLVM IR after it compiles user codes. 