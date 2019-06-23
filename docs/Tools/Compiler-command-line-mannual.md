# Command line arguments for compiler (birdeec)

Birdee compiler can be invoked by command line. The executable file of Birdee compiler should be "birdeec" or "birdeec.exe".

The accepted arguments are listed in the following table. Note that the "[..]" means a user input string.

|Arugment|Alias|Optional?|Explanations|
|---|---|---|---|
| -i [file_name]| N/A | Required | The input source file path|
| -o [file_name]|  N/A | Required | The output file path|
| -s | --script | Optional | Treat the input source file as a python generative script |
| -e  | N/A | Optional | Put the top-level code in a function called "main" instead of "{modulename}.main"|
| N/A | --corelib | Optional | Prevent the compiler from auto importing the core Birdee modules for the source code|
| N/A | --printir | Optional | Print the LLVM IR to stderr after compilation |
| -p [prefix] | N/A | Optional | Prepend a prefix for each global symbols of the compiled object file |
| N/A | --print-import | Optional | Don't compile the whole file. Print the imported module names and currently compiled module to stdout only. Cannot be used with "-s" switch. "-o" switch can be omitted when using this switch |
| -O [optimize-level] | N/A | Optional | Set the optimization level of the compiler, optimize-level can be 0~3. "-O 0" means no optimization. The larger optimize-level, the more aggressive optimization. By default to be 0 |
| --llvm-opt-start ... --llvm-opt-end | N/A | Optional | Give the arguments between "--llvm-opt-start" and "--llvm-opt-end" to the LLVM optimizer. |