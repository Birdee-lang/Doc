# 5. Array

## 5.1 Array basics

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
dim ptr as pointer = arr.get_raw()                   #arr.get_raw() gets the native pointer of the array
```

## 5.2 Array initializers

You can create an array and assign inital values of the elements at the same time by array initializers. The syntax is

```vb
[expr1, ...]
```

An array initializer is composed of a pair of "\[\]", and between the brackets, there should be one or more expressions, sparated by commas. An array initializer itself is an expression of array type, and the contents of the resulting array is initialized by the expressions between the brackets. Here are some examples:

```vb
dim a as int[] = [1,2,3,4]
dim b = ["hi","hello"]
println(b[0]) #should be "hi"
```

The compiler will automatically infer the type of the array initializer by the expressions in the brackets. In the above example, the first line's array contains 4 integers, so it is a integer array. In the second line, the two elements of the array are strings, so the variable "b" is automatically infered as `string[]`.

When the types of the expressions in the array initializer are not the same, the compiler will try to find the most "general" type by the expression types in the array. The rules to find a general type can be found in the "Auto type conversion" in "Basics" chapter. If the types of expressions are incompatable (or cannot be automatically converted), an error will be raised.