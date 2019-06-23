# 7. Modules and Imports

Birdee programs can be encapsulated into modules. A module is a collection of functions, classes and global variables that can be invoked and used by other Birdee code. A similar concept for Birdee's module is the object files in C/C++ compilation systems. The main difference is that Birdee's modules contains metadata to describe the prototype of the exported functions, the definitions of the classes and the types of the exported variables in the module, while in C/C++, an object file is just a collection of code and variable, and users must write a header file or explicitly declare the variables and functions with "extern" keyword. Such header files are no longer needed in Birdee.

When a Birdee program is compiled by the "birdeec" compiler, the source code will be converted into a module, which consists of an object file (which can be linked with C/C++ object files) and a metadata file (\*.bmm file. "bmm" represents "Birdee Module Metadata") which describes the variables, classes and functions of the module. To use a module that already exists, users can use "import" keyword in the Birdee code. Note that a module can "import" multiple existing modules and all "import" should be written at the beginning of the Birdee code before any other top level code. The functions, classes and variables in the top-level of a module are called symbols. After importing a module, the symbols can be used in the current Birdee module. 

Note that if a module is imported by any syntax above, the top-level code of the imported library will be executed exactly once.

There are 4 ways to import an external module.

## 7.1 Module Import

The first form of syntax is:

```java
import {module_name}
```

The "module_name" is composed of directory names separated by dots ("."). The Birdee compiler will search the local file system for the module to be imported using the path provided by the "module_name". There are two ways setting the root path of searching the modules. One way is to set the "BIRDEE_HOME" environment variable in your OS. The compiler will search "BIRDEE_HOME/blib" for the target module. Another way is to set a module search path in the command line arguments for "birdeec" with "-l" switch. The compiler will first try to find the target module with root directory set by the command line arguments, then by the "BIRDEE_HOME" environment variable. For example, if the "BIRDEE_HOME" is set as "/home/menooker/birdee" and we have a "-l /sourcecode/mylib" argument in the command line, to resolve the code "import sys.net.socket" (let's assume there exists such a module), the compiler will first find "/sourcecode/mylib/sys/net/socket.bmm". If it does not exist, "/home/menooker/birdee/blib/sys/net/socket.bmm" will be searched.

After importing with this syntax, users can use "{module_name}.{symbol_name}" to access the variables, functions or classes in the module. For example, 
```vb
import sys.net.socket
sys.net.socket.connect("192.168.0.1",999)
```

The "{module_name}.{symbol_name}" is called qualified name of a symbol. However, the module we are talking about here has not been implemented yet unfortunately.

## 7.2 Symbol Import

The second form of syntax is:

```java
import {module_name}:{symbol_name}
```

This form of import will do "module import" just as the last section does. In addition, it will import a symbol name of a variable, function or class from the target module. Users can simply use "symbol_name" to access the the symbol instead of the long qualified name. For example

```vb
import sys.net.socket:connect
connect("192.168.0.1",999)
```

## 7.3 All Symbols Import

All symbols import will do "Symbol Import" on all symbols of the target module. The syntax is:

```java
import {module_name}:*
```

After all symbols import, the variables, functions and classes can be accessed using the names instead of qualified names. For example

```vb
import sys.net.socket:*
connect("192.168.0.1",999)
```

## 7.4 Auto Import

The compiler will automatically import all symbols from some core modules of Birdee, such as "birdee.bmm". "birdee.bmm" contains the core features like "string", "println".

## 7.5 Private symbols

As we have discussed above, the classes, variables and functions (include function declartions) in the top-level will be exported in a Birdee module, which can be used by other modules. Sometimes, developers do not want to expose the internal classes/variables/functions of a module. In this case, they can add "@private" before the symbols (classes/variables/functions) that they want to hide from other modules. If a symbol is marked "private", other modules cannot access it by importing the module. Here is an example of a module with private symbols:

```vb
@private dim private_v = 1 #the "private" can be followed by a space

@private
class someclass #the "private" can be followed by a newline

end

function foo() #by default, it is public
end
```