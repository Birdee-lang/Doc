# 4. Scope and names

We first introduce a concept: basic block. A basic block is a block of code. The basic block in the highest-level is the top-level code; A simple "if-else" has two basic blocks: one for condition being true and the other for condition being false; A for loop itself is a basic block. As you can see, basic blocks may have some children basic blocks.

Scopes define where a name (variable/function/class) can be used and is visible to the programmers. Every basic block has one corresponding scope. When a name is defined, the name will be added into the current basic block's scope. Scopes are maintained in a tree, where a parent scope may have zero or multiple children scopes. There are some simple rules for scopes in Birdee:
 1. The code in a child scope can reference all names (for variable/function/class) of its ancestor scopes - In a child basic block, you can use the functions/variables/classes defined in any ancestor basic blocks.
 2. The code in a parent scope is unaware of the names in a child basic block - In a parent basic block, you cannot use the variables (or functions, etc.) which is defined in a child basic block. You can define a variable in the parent basic block which have the same name of some variable in the child basic block.
 3. In the same scope, defining two variables/functions/classes with the same name is not allowed.
 4. In the child scope, it is allowed to define a variable (or function/class) which has the same name of some variables (or functions, etc.) in the ancestor scopes.
 5. When resolving a name, if the name is defined in some ancestor scopes or in the current scope, the variable (or function, etc.) defined in the lowest level of scope will be selected.

Note that the parameters of functions can be viewed as defined variables in the function's scope.

For more explanations, see comments:

```vb
dim a as int = 2
function compute(b as int) as int
	return a*b  # Okay because of rule 1
end

b=3 # Won't compile! Due to rule 2
if a==2 then
	dim c as float=123.2
end
c=333.0 # Won't compile! Due to rule 2

dim a as string # Won't compile! Due to rule 3

function show(b as int)
	dim a = "hi"  # Okay because of rule 4
	println(a + int2str(b))  # Okay because of rule 5, the definition in the current scope is selected
end
```

## 4.1 Name resolution rules

The name resolution rules defines how identifiers are resolved and bound to the function/variable definition. Given a name of identifier for expression, Birdee language should find the name in the definitions in the following order, from upper ones to lower ones. Once an definition name is matched with the identifier name, the compiler will bind this identifier expression with the selected definition, and will disregard all definitions in lower orders.

1. The names of local variables in the current basic block
2. The names of local variables (including function arguments) in the ancestor basic blocks (nearest first)
3. The names of template arguments (we will discuss templates later. Note that compilers only search constant expression template arguments here.)
4. (If currently in a member function of a class,) The field names of the current class
5. (If currently in a member function of a class,) The member function names of the current class.
6. The global variable (defined in top-level) names defined in the current module
7. The functions names defined in the current module
8. The imported global variable (defined in top-level) names defined in other modules
9. The imported functions names defined in other modules
10. The imported package names

The above is the order of name resolution of identifier expressions. Birdee also defines the resolution order for type names. The following order applies to types (e.g. the identifiers after "as"):

1. The template argument names of the current function
2. The template argument names of the current class (if exists)
3. The "functype" or closure names of the current module
4. The imported "functype" or closure names of the imported modules
5. The class names of the current module
6. The imported class names of the imported modules
