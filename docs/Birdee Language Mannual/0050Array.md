# 5. Array

An array is an sequential collection of values with the same type. To use an array type, you can append "[]" to other types and you will get a array of that type. See example:

```vb
dim arr1 as int[]
```

Here "arr1" is a variable of integer array type.

Arrays of objects are also allowed:

```vb
dim arr2 as string[]
``` 

You can declare arrays of arrays (multi-dimension array). Here is an example of two-dimension array:

```vb
dim arr2 as float[][]
```

To use an array, you should first allocate space for it, by "new" operator. The syntax is:

```vb
new {type} * {number_of_elements} 
```

For example, to create an array of integer with 10 elements:

```vb
dim arr as int[] = new int * 10
```

To create an [10x20] array of float, you can:

```vb
dim arr2 as float[][] = new float * 10, 20
```

Finally, you can access the array by "[ ]":

```vb
arr[0]=12
dim c = arr[0] + 23
arr2[1][3]=3.14
```

Note that surrounded by "[ ]" is the index of the array element you want to access. The index must be an integer type (int, uint, long, ulong, ...). Also, the index starts at 0 and ends at number_of_elements-1. If you create an array by "new int[10]", the valid index will be within [0 ~ 9].

The array variable is a reference to the array, not the array itself. So the following code:

```vb
dim arr as int[] = new int * 10
dim arr2 as int[] = new int * 10
arr2 = arr
```

will not copy the array of 10 elements to "arr2", but make the variable "arr2" points to the same array of "arr". So if we assign a value to an element of "arr", you can find the change by the variable "arr2", because both variables points to the same array!

```vb
dim arr as int[] = new int * 10
dim arr2 as int[] = new int * 10
arr2 = arr
arr[3] = 123
println(int2str(arr2[3])) # you will see "123"
```

Also note that variables for classes are also references. The same effect applies to class variables too.

You can access some properties of an array by:

```vb
dim arr as int[] = new int * 10
println("The number of elements is " + arr.length()) #arr.length() gets the # of elements
dim ptr as pointer = arr.getRaw()                    #arr.getRaw() gets the native pointer of the array
```
