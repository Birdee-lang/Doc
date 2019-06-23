# The automatic building tool for Birdee - bbuild

The Birdee compiler "birdeec" only generates object files from Birdee source code, but it does not help you to link it. In large projects, the dependencies among modules could be complicated. Without bbuild, users may resort to Makefile or other building tools to compile and link such projects, which involves manually writing Makefiles or other configuration files to tell the building tool about the dependencies. bbuild aims to free the users from manually managing the building configuration by automatically finding and compiling the dependencies. In addition, bbuild can optionally link the program with all dependencies.

## Requirements

bbuild is a Python3 program. It depends on native compiler tool sets: On Linux, it depends on gcc; On Windows, it depends on MSVC. Make sure these dependencies are installed in your system before using bbuild. To use the linker of MSVC, you need to run bbuild in the Visual Studio x64 Commmand Prompt.

## What does bbuild do?

You should specify a "root" module to compile. Given a "root" module, bbuild will find all modules the root module directly or indirectly depends on. For all dependent Birdee modules, bbuild first finds any compiled object file of the dependent module. If no compiled object file is found, bbuild searches the Birdee source code using the module name and compiles the found module source code into object file. Finally, if the user requires to link the root module with all dependent modules, bbuild will call the native linker (linker.exe on Windows or gcc on Linux) to link all object files.

Like building tools including Makefile, bbuild will check the timestamps of the source code files and the resulting binary files. It will only re-compile the modified source code files since the last build, and skip the unchanged source files. 

## How to use bbuild?

```bash
python $BIRDEE_HOME/pylib/bbuild.py -i {source_dir} -o {out_dir} [-bs {binary_file_search_path}] [-le {link_executable_target}] [-lc "{link_command}"] [-j {thread_num}] root_module1 [other_root_modules...]
```

 * -i: the source root dir
 * -o: the output object file root dir
 * -le: link to an executable. The path to the link target
 * -bs: binary \*.bmm and \*.obj/\*.o file search root dir. The out\_dir specified by "-o" switch is automatically added to the binary search path
 * -lc: The extra commands to pass to the linker. It is recommended to add a pair of "" around link\_command
 * -j: set the number of parallel compilers 
 * root_modules: a list of the module names, e.g. org.bird.quail. bbuild will find source from source_root_dir/org/bird/quail.{txt/bdm}. It will also resolve any direct or indirect dependencies of it.

 You can specify multiple search directories by adding multiple "-i", "-bs" switches. For any dependent module, bbuild will first search the paths given by "-bs" switch. e.g. Given "-o /home/menooker/bin -bs /home/menooker/birdee" and a module "org.bird.quail", bbuild will search for file "/home/menooker/birdee/org/bird/quail.bmm" or "/home/menooker/bin/org/bird/quail.bmm". If the binary file is not found, bbuild will try to compile it from the source code. Given "-i /home/menooker/project", it will search for source file "/home/menooker/project/org/bird/quail.{txt,bdm}". The compiled binary of module will be at "/home/menooker/bin/org/bird/quail.bmm".

 For example, let's compile the main module test\_package.b.mod2 (The code can be found in "tests" of Birdee source code). The commands and outputs are shown below

```
menooker@DESKTOP:~/Birdee/tests$ cat test_package/b/mod2.txt
package test_package.b

import test_package.a.mod1:func
import logic_obj_cmp:a

func()
println(a)
menooker@DESKTOP:~/Birdee/tests$ find test_package/*
test_package/a
test_package/a/mod1.txt
test_package/b
test_package/b/functype_test_lib.txt
test_package/b/mod2.txt
menooker@DESKTOP:~/Birdee/tests$ ls logic_obj_cmp.txt
logic_obj_cmp.txt
menooker@DESKTOP:~/Birdee/tests$ python3 $BIRDEE_HOME/pylib/bbuild.py -i . -o linuxbin test_package.b.mod2 -le linuxbin/mod2 -lc "-g -lpthead"
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./test_package/b/mod2.txt --print-import
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./test_package/a/mod1.txt --print-import
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./logic_obj_cmp.txt --print-import
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./test_package/a/mod1.txt -o linuxbin/test_package/a/mod1.o -l /home/menooker/Birdee/BirdeeHome/blib -l . -l linuxbin
Wrote linuxbin/test_package/a/mod1.o
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./logic_obj_cmp.txt -o linuxbin/logic_obj_cmp.o -l /home/menooker/Birdee/BirdeeHome/blib -l . -l linuxbin
Wrote linuxbin/logic_obj_cmp.o
Running command /home/menooker/Birdee/BirdeeHome/bin/birdeec -i ./test_package/b/mod2.txt -o linuxbin/test_package/b/mod2.o -l /home/menooker/Birdee/BirdeeHome/blib -l . -l linuxbin -e
Wrote linuxbin/test_package/b/mod2.o
Running command gcc -o linuxbin/mod2 -Wl,--start-group /home/menooker/Birdee/BirdeeHome/lib/libBirdeeRuntime.a /home/menooker/Birdee/BirdeeHome/blib/birdee.o linuxbin/test_package/a/mod1.o linuxbin/logic_obj_cmp.o linuxbin/test_package/b/mod2.o -lgc -Wl,--end-group -g -lpthead
menooker@DESKTOP:~/Birdee/tests$ find linuxbin/*
linuxbin/logic_obj_cmp.bmm
linuxbin/logic_obj_cmp.o
linuxbin/mod2
linuxbin/test_package
linuxbin/test_package/a
linuxbin/test_package/a/mod1.bmm
linuxbin/test_package/a/mod1.o
linuxbin/test_package/b
linuxbin/test_package/b/mod2.bmm
linuxbin/test_package/b/mod2.o
```