# 1. Hello world!

Our first Birdee program prints a line of words "Hello world" on the console. Create a file named "hello.bdm", and write a simple line of code in the file:

```vb
println("Hello world")
```

Then switch to the directory of the file "hello.bdm" and compile it with command in Linux:

```shell
python $BIRDEE_HOME/pylib/bbuild.py -le ./hello -i . -o . hello
```

If you are using Windows, you should run the following command in Visual Studio x64 Commmand Prompt. (You can find it in the Visual Studio directory of the start menu.)

```shell
python %BIRDEE_HOME%\pylib\bbuild.py -le .\hello.exe -i . -o . hello
```

"bbuild.py" is a high-level tool for Birdee source compiling and linking. You should use Python 3 to run this command. The above command tells the compiler to find the source code in the current directory and put the object files in the current directory too. It also links the object files to an executable binary file at "./hello".

Now we have our first Birdee executable "hello". Run it by:

```shell
./hello
```

Instead of using bbuild, you can also manually compile and link the program with commands:

```shell
$BIRDEE_HOME/bin/birdeec -i hello.bdm -o hello.o -e
gcc -o hello hello.o $BIRDEE_HOME/blib/birdee.o $BIRDEE_HOME/lib/libBirdeeRuntime.a -lgc
```
 
More command line options for "birdeec" can be found [here](../Tools/Compiler-command-line-mannual.md). "birdeec" is the core compiler for Birdee language and it generates object files only. You need to link the object file with other necessary files to generate an executable. Fortunately, our tool bbuild can do these all for you. More details of bbuild can be found [here](../Tools/bbuild.md)

An simpler way to try and run Birdee code without compiling and linking is to use the Birdee playground. It provides a interactive environment which accepts Birdee code and immediately evaluate the result of it after you press "Enter". More on the playground can be found [here](../Tools/Birdee-playground-(REPL).md)