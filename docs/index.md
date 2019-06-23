# Welcome

Birdee is a new programming language with C++-level performance and with light-weighted & managed runtime. 

## Language mannual

Please follow the links in the navigation bar.

## Getting Birdee 
### Building Birdee from scratch

Please follow the instructions in the [link](https://github.com/Birdee-lang/Birdee2/blob/master/BUILDME.md).

### Pre-compiled Birdee

Please follow the [link](https://github.com/Birdee-lang/Birdee2/releases) to download the pre-built Birdee.

For Windows version, just unzip and add the unzipped directory to the environment variable as `BIRDEE_HOME`. In the directory pointed by `BIRDEE_HOME`, there should be directories named `bin`, `pylib`, `blib` and `src`.  Also, Birdee compiler depends on a specific version of Python (e.g. 3.7).

For Linux Debian systems, use `sudo dpkg -i birdee_xxxx.deb` to install and `sudo dpkg -r birdee` to remove. This package requires libpython3.7-dev, libgc-dev and LLVM-6.0 to be installed. Remember to add `/usr/bin/birdee0.1` to the environment variable as `BIRDEE_HOME`.

## What's in the Birdee toolkit?

Birdee provides several useful tools to help develop Birdee programs.

### Birdee compiler

The compiler compiles a single Birdee module source file (\*.bdm) into an native object file (\*.o or \*.obj) or LLVM IR file. The outcome of the compiler can be further linked into an executable file by external linkers (e.g. ld on Linux, link.exe of MSVC). Please note that Birdee compiler alone will not help you link the Birdee modules. You should link the modules by yourself or by the auto-building tool "bbuild". For more information on the compiler command line arguments, please refer to [here](Tools/Compiler-command-line-mannual.md).

Besides the object files, the Birdee compiler will generate an meta-data file (\*.bmm), for the compiled module, which is in JSON format. It contains information on the exported variables, classes and the function definitions of the module. This overcomes the issue that native object files will lose the important meta-data of the source code.

### Birdee auto-building tool - bbuild

bbuild is a Python-based auto-building tool provided by Birdee, to compile multiple Birdee modules and optionally link them into an executable/shared object file. It can

 * Parse the `import` dependency of the Birdee modules, and automatically find the source code (or compiled object files) of dependent modules.
 * Automatically compile the modules from the source files.
 * Optionally link the object files into an executable/shared object file

For more information on bbuild, please follow the [link](Tools/bbuild.md).

### Birdee playground (REPL)

To help to learn Birdee language and try Birdee programs, we provide a playground program, where you can simply type Birdee code and instantly see the results without compiling the whole program using the compiler. For each line of code you input, the playground will compile and execute it and prints the execution results on the console. Please follow the [link](Tools/Birdee-playground-(REPL).md) to try it.

## Tips on running compiled Birdee programs

On Linux, a compiled Birdee application has an additional dependency on libgc. Please make sure it is installed on the system where the application runs.

### Run Birdee programs on Windows!

To run Birdee programs that are compiled with Birdee compiler on Windows, make sure "gc_64.dll" is in DLL search path (e.g. where the ".exe" file is). If you use exceptions in your Birdee program, make sure "libgcc_s_seh-1.dll" and "libwinpthread-1.dll" are also in DLL search path.

If you use the pre-compiled binaries of Birdee, these DLLs can be found in the "bin" directory. These DLL files can also be downloaded via the following links. [BaiduYun](https://pan.baidu.com/s/1FWnHpQkxj5PC4DP1PEMRlg) or [GoogleDrive](https://drive.google.com/open?id=1GIH-YDe2IFMnaYE91uOXAJlnXJjX3PvT). Note that "libgcc_s_seh-1.dll" and "libwinpthread-1.dll" are extracted from mingw64.

