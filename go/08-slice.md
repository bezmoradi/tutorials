# Go > Array & Slice

Go offers two types of similar data structures for storing a list of items:

-   **Array**: A numbered sequence of similar-type elements which is fixed in length
-   **Slice**: An array that can grow in size or shrink

The key point here is that both Array and Slice elements must be of a **single** type; meaning all members must be either `int`, `string`, or any other type. As arrays are fixed in size, they are mostly used for Go internals and in general, slices are more common.

## Array

The syntax to create an array is as follows:

```go
package main

import "fmt"

func main() {
	numbers := [4]int{1, 2, 3, 4}
	fmt.Println(numbers)
}
```

`[4]int` defines the type of the values which in this case is an array of integers and `4` defines the size of the array. Another way of defining the length of an array is the `...` as follows:

```go
numbers := [...]int{1, 2, 3, 4}
```

`...` indicates that the size of the array should be determined by the **number of elements** provided in the **initialization**. Using `...` can be helpful when you want to declare an array and have the compiler infer the size based on the provided elements, ensuring that the array is exactly large enough to hold those elements. If our array is going to hold lots of elements and it's easier to place each element on a single line, we have to do it this way:

```go
numbers := [4]int{
	1,
	2,
	3,
	4,
}
```

The **key** thing here is the trailing comma that comes after the last element (`4,`) which is **mandatory** otherwise we will get compile-time error:

```text
syntax error: unexpected newline in composite literal; possibly missing comma or }
```

We can also first initialize the array then add elements to it:

```go
var numbers [4]int
fmt.Println(numbers)
```

If we see the output of the above program, `[0 0 0 0]` will be printed and as the null value for `int` type is zero, we see four 0s. The visual representation of the `[4]int` array is as follows:

```text
+-------+-------+-------+-------+
|       |       |       |       |
|   0   |   0   |   0   |   0   |
|       |       |       |       |
+-------+-------+-------+-------+
```

To populate an existing array we can:

```go
numbers = [4]int{1, 2, 3, 4}
```

Previously we said that if we initialized an array without adding any values right off the bat, based on the type of the array, the array will be filled with the default value for the type. Now let's consider the following scenario:

```go
var numbers [4]int
numbers[3] = 4
fmt.Println(numbers) // Prints [0 0 0 4]
```

As we have decided to only populate the last index (`3` which maps to the fourth element), the first three elements will hold the default value (`0`) and the last element will hold `4`. We can also do it this way:

```go
var numbers [4]int
numbers[0] = 1
numbers[1] = 2
numbers[2] = 3
numbers[3] = 4
```

To access an element of an array we can:

```go
fmt.Println(numbers[3]) // Prints 4
```

If we try to access an index which does not exist, both the IDE warns us and the compiler shows an error as follows:

```go
numbers := [4]int{1, 2, 3, 4}
fmt.Println(numbers[4]) // invalid argument: index 4 out of bounds [0:4]
```

To know more on this topic, refer to [Index out of Range Error](#index-out-of-range-error). To update a specific index of the array with a new value we can:

```go
numbers := [4]int{1, 2, 3, 4}
numbers[3] = 10
fmt.Println(numbers) // Prints [1 2 3 10]
```

Now in order to write a test for an array, let's first refactor our code to move the array logic into its own function:

```go
package main

import (
	"fmt"
)

func sum(numbers [4]int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}

	return sum
}

func main() {
	numbers := [4]int{1, 2, 3, 4}
	fmt.Println(sum(numbers))
}
```
We have a `sum` function which receives an array of integers as input param and returns the sum of all values. To test the `sum` function, let's create a test file called `main_test.go` as follows:

```go
package main

import "testing"

func TestSum(t *testing.T) {
	got := sum([4]int{1, 2, 3, 4})
	want := 10
	if got != want {
		t.Errorf("expected %v but received %v", want, got)
	}
}
```

## Slice

The only difference between creating an array and a slice is that we need to omit the hard-coded length:

```go
numbers := []int{}
```

Another way to create an empty slice is:

```go
var numbers []int
```

If you try to print the `numbers` slice, you would get `[]`:

```go
numbers := []int{}
fmt.Println(numbers) // Prints []
```

To add new elements to a slice we can use the `append` function as follows:

```go
numbers := []int{}
numbers = append(numbers, 1)
numbers = append(numbers, 2)
numbers = append(numbers, 3)
numbers = append(numbers, 4)
fmt.Println(numbers) // Prints [1 2 3 4]
```

Keep in mind that instead of the `:=` operator we have used the `=` which is used for assigning a value. The `append()` function returns a new slice header. If the slice has sufficient capacity, `append` reuses the existing backing array and adds the new element. However, if capacity is exceeded, `append` allocates a new, larger backing array, copies all existing elements, and then adds the new element. This is why you must always assign the result of `append` back to your slice variable. We can append multiple values in one go:

```go
numbers = append(numbers, 1, 2, 3, 4)
```

Basically, the length of an empty slice is `0`; that's why, unlike an array, we cannot access indexes like so:

```go
numbers := []int{}
numbers[0] = 1
```

In the above case, we would get the following error:

```text
panic: runtime error: index out of range [0] with length 0
```

If you're interested in creating a slice with non-zero length, you can use the `make` function as follows:

```go
numbers := make([]int, 4)
fmt.Println(numbers) // Prints [0 0 0 0]
```

The visual representation of the above slice is as follows:

```text
+-------+        +-------+-------+-------+-------+
|       |        |       |       |       |       |
|pointer|------->|   0   |   0   |   0   |   0   |
|       |        |       |       |       |       |
+-------+        +-------+-------+-------+-------+
|  len  |
|   4   |
|       |
+-------+
|  cap  |
|   4   |
|       |
+-------+
```

Now we can access indexes and populate them like we did with arrays:

```go
numbers := make([]int, 4)
numbers[0] = 1
numbers[1] = 2
numbers[2] = 3
numbers[3] = 4
fmt.Println(numbers) // Prints [1 2 3 4]
```

You need to be cautious when you want to use the `append` function while you have created your slice using the `make` function:

```go
numbers := make([]int, 4)
numbers = append(numbers, 1)
numbers = append(numbers, 2)
numbers = append(numbers, 3)
numbers = append(numbers, 4)
fmt.Println(numbers) // Prints [0 0 0 0 1 2 3 4]
```

Basically, what the above code does is that `make` has created a slice with four elements with default values of `0` then `append` will add new values to the next index.  
We can use the `copy` function like this:

```go
numbers := []int{1, 2, 3, 4}
slice := make([]int, 2)
copy(slice, numbers)
fmt.Println(slice)
```

`slice` is a slice of integers with the length of `2` and what `copy` does is that it copies the first two elements of the source variable (`numbers`) into the destination (`slice`).


### An Intro to `len()` and `cap()` Built-in Functions

The `len()` function shows the length of an array or slice:

```go
numbers := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
fmt.Println(len(numbers)) // Returns 10
```

The `cap()` function which is short for **Capacity** returns the capacity of the array or slice; in the following example, as the capacity of the array is 10, obviously it returns 10:

```go
numbers := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
fmt.Println(cap(numbers)) // Returns 10
```

If two arrays are of the same length, we can assign one to the other:

```go
func main() {
	numbers1 := []int{1, 2, 3}
	numbers2 := []int{10, 20, 30}
	numbers1 = numbers2
	fmt.Println(numbers1) // Returns [10 20 30]
}
```

### How to Get A Slice of An Array

To specify the start and end of the slice we can follow the `[inclusive:exclusive]` format:

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers[2:7] // [inclusive:exclusive]
fmt.Println(numbers)  // Returns [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice)    // Returns [2 3 4 5 6]
```

In `numbers[2:7]` index 2 which holds number two is included and index 7 which holds number seven is NOT included; that's why we have `[2 3 4 5 6]`.

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers[:6] // [:exclusive]
fmt.Println(numbers) // [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice) // [0 1 2 3 4 5]
```

In the above snippet, `[:6]` means from the very beginning of the slice to the element at index 6 (it's not included) so we have `[0 1 2 3 4 5]`.

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers[4:] // [inclusive:]
fmt.Println(numbers) // [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice) // [4 5 6 7 8 9]
```

In the above case, `[4:]` can be translated to `[inclusive:]` meaning from the element at index 4 which is number four (including itself) to the end of the slice which will be `[4 5 6 7 8 9]`.

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers[:] // [:]
fmt.Println(numbers) // [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice) // [0 1 2 3 4 5 6 7 8 9]
```

`[:]` means from the element at index 0 including itself to the end of the slice; so we get `[0 1 2 3 4 5 6 7 8 9]`. An equivalent of the above code is:

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers
fmt.Println(numbers) // [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice)   // [0 1 2 3 4 5 6 7 8 9]
```

Technically speaking, a slice is a window to the original array; in other words, it does not copy anything from the original array but contains a pointer to part of that array that's why mutating a slice element would result in mutation in the original array/slice as well. For more details on this behavior, see [How Slices Behave When Passed to Functions](#how-slices-behave-when-passed-to-functions).

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := numbers
slice[0] = 1000
fmt.Println(numbers) // [1000 1 2 3 4 5 6 7 8 9]
fmt.Println(slice) // [1000 1 2 3 4 5 6 7 8 9]
```

In the above example, we are mutating the first element of the `slice` at index 0 and if we print both the original slice (`numbers`) and the new one (`slice`), we'll see that they are both mutated. To copy a slice/array referring to a pointer in memory, we can:

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := make([]int, len(numbers))
copy(slice, numbers)
slice[0] = 1000
fmt.Println(numbers) // [0 1 2 3 4 5 6 7 8 9]
fmt.Println(slice)  // [1000 1 2 3 4 5 6 7 8 9]
```

When you use the copy function in Go, it creates a new slice and copies the elements from the source slice to the destination slice. The new slice is completely independent of the source slice, and modifications to one do not affect the other.

### How to Create Multidimensional Array (Slice)

The notation `[][]int` signifies a slice (`[]`) capable of accommodating an arbitrary number of elements, all of which are of type `[]int`:

```go
odds := []int{1, 3, 5, 7, 9}
evens := []int{0, 2, 4, 6, 8}
total := [][]int{odds, evens}
fmt.Println(total) // [[1 3 5 7 9] [0 2 4 6 8]]
```

We can optionally add `[2][]int{odds, evens}` meaning the length of our slice is `2`. For example, if we change the initial length from `2` to `3` we have:

```go
[3][]int{odds, evens} // [[1 3 5 7 9] [0 2 4 6 8] []]
```

Basically, it adds an empty slice to the list.

### How to Remove An Element from Slices

There isn't any build-in function like `append()` to remove elements! The workaround though for this scenario is as follows:

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
firstElementRemoved := numbers[1:]
fmt.Println(firstElementRemoved) // [1 2 3 4 5 6 7 8 9]
```

We have actually removed the first element from the slice. To remove the last element we have:

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
lastElementRemoved := numbers[:len(numbers)-1]
fmt.Println(lastElementRemoved) // [0 1 2 3 4 5 6 7 8]
```

In the following example, we want to remove number 5 from the slice

```go
numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
numberFiveRemoved := append(numbers[:5], numbers[6:]...)
fmt.Println(numberFiveRemoved) // [0 1 2 3 4 6 7 8 9]
```

## How Slices Behave When Passed to Functions

In Go, **everything is passed by value**—there is no pass by reference. However, slices exhibit interesting behavior because of their internal structure.

A slice is internally a three-field struct containing:
1. A pointer to the underlying array
2. The length (len)
3. The capacity (cap)

When you pass a slice to a function, Go passes a **copy of this slice header by value**. However, since the slice header contains a pointer to the underlying array, both the original and the copied slice header point to the **same backing array**. This means modifications to array elements are visible to both:

```go
package main

import "fmt"

func main() {
	numbers := []int{1, 2, 3, 4}
	fmt.Println(numbers) // [1 2 3 4]
	update(numbers)
	fmt.Println(numbers) // [0 2 3 4]
}

func update(n []int) {
	n[0] = 0
}
```

When you pass the `numbers` slice to the `update` function, the slice header is copied, but the pointer inside that header still points to the same underlying array. This is why modifying `n[0]` affects the original `numbers` slice—they share the same backing array.

**Important caveat**: If the function uses `append` and causes the slice to grow beyond its capacity, a new backing array is allocated, and the connection to the original slice's array is broken. The original slice remains unchanged because only the copied slice header gets updated with the new array pointer.

## How to Create A Slice of Structs

To do that we have:

```go
package main

import "fmt"

type product struct {
	id    int
	title string
}

func main() {
	products := []product{
		product{id: 1, title: "First"},
		product{id: 2, title: "Second"},
	}
	fmt.Println(products) // [{1 First} {2 Second}]
}
```

The IDE would warn that the `product` keyword in an element like `product{id: 1, title: "First"}` is redundant just because Go would figure out that the underlying type of that element is `product`. So we can simplify the above code as follows:

```go
products := []product{
	{id: 1, title: "First"},
	{id: 2, title: "Second"}, // Comma must exist
}
```

Again, an important thing to remember is the trailing comma after the second element. We can also first initialize the variable then add elements to it:

```go
var products []product
products = append(products,
	product{id: 1, title: "first"},
	product{id: 2, title: "second"},
)
fmt.Println(products) // [{1 first} {2 second}]
```

## How to Use The `...` Operator

So far, we have managed to use the `append` function a couple of times. The `append()` function can also be used for merging two arrays/slices:

```go
odds := []int{1, 3, 5, 7, 9}
evens := []int{0, 2, 4, 6, 8}
total := append(odds, evens...)
fmt.Println(total) // [1 3 5 7 9 0 2 4 6 8]
```

The key point here is that if we need to pass an array/slice as the second argument to the `append()` function, we have to use the special syntax `...` which spreads the elements like so:

```go
odds := []int{1, 3, 5, 7, 9}
total := append(odds, 0, 2, 4, 6, 8)
fmt.Println(total) // [1 3 5 7 9 0 2 4 6 8]
```

## Use The `make()` function to Create Slices

The `make` function initializes a slice with a specified length and capacity. It's useful when you know the initial size of the slice you need, and optionally, the capacity to which it might grow without having to dynamically resize as elements are appended:

```go
package main

import "fmt"

func main() {
	slice := make([]int, 5, 10)
	fmt.Println(slice) // [0 0 0 0 0]
}
```

This creates a slice with the length of 5 and capacity of 10. Now if we change the above program to:

```go
slice := make([]int, 0, 10)
fmt.Println(slice) // []
```

Basically, the main difference lies in the fact that when we added 5 as the second argument to the `make()` function, we asked that function to initialize the slice and as the default value for `int` type is `0`, we have `[0 0 0 0 0]`. In the second case, when we passed `0` as the second argument, it means that we do not want to initialize the slice that's why we get `[]`.  
Likewise, the slice literal `[]int{}` initializes an empty slice. This method is suitable when the size or capacity of the slice isn't predefined, and you plan to append elements dynamically based on your program's logic.  
Technically, you can use `make()` when you know the initial size and optionally the capacity of the slice, especially in scenarios where you want to allocate contiguous memory for the slice elements upfront or when dealing with larger slices to avoid **frequent reallocations** (because every time we add a new element without sufficient capacity, behind the scenes a new array is created).

```go
numbers := make([]int, 2, 10)
numbers = append(numbers, 2)
fmt.Println(numbers) // [0 0 2]
```

Using `make()` function as above is more memory-efficient because each time we add an element, Go does not have to allocate new space because already we have enough space for ten elements and only it will start adding new space when we go beyond 10 elements. As shown above, the first two slots are initialized to their default values (indexes 0, 1) and the `append()` function starts adding after those initialized slots which in this case is index 2. In order to fill in the first two slots we have:

```go
func main() {
	numbers := make([]int, 2, 10)
	numbers[0] = 1
	numbers[1] = 2
	numbers[2] = 3 // runtime error: index out of range [2] with length 2
}
```

We would get error because there is no index of 2 in our slice. To add elements in this case, we need to use the `append()` function:
```go
numbers := make([]int, 2, 10)
numbers[0] = 1
numbers[1] = 2
numbers = append(numbers, 3)
fmt.Println(numbers) // [1 2 3]
```
Something good to know about the `make()` function is that in the following example `make([]int, 0, 10)` created a slice with the capacity of 10 and when we add numbers 11 and 12 to our slice, Go's runtime uses a growth strategy that depends on the current capacity. For slices with capacity less than 256, the capacity typically doubles. For slices with capacity of 256 or more, the capacity grows by approximately 1.25x (though the runtime may round up to optimize memory allocation). In this case, when we exceed capacity 10, it grows to 20, and when we exceed 20, it grows to 40:

```go
numbers := make([]int, 0, 10)
numbers = append(numbers, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
fmt.Println(numbers, len(numbers), cap(numbers))
fmt.Println("----------")
numbers = append(numbers, 11, 12)
fmt.Println(numbers, len(numbers), cap(numbers))
fmt.Println("----------")
numbers = append(numbers, 13, 14, 15, 16, 17, 18, 19, 20, 21)
fmt.Println(numbers, len(numbers), cap(numbers))
```

In the output we have:

```text
[1 2 3 4 5 6 7 8 9 10] 10 10
----------
[1 2 3 4 5 6 7 8 9 10 11 12] 12 20
----------
[1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21] 21 40
```

## Advanced Slice Concepts

### Nil Slices vs Empty Slices

While they both have zero length, nil slices and empty slices are not the same:

```go
package main

import "fmt"

func main() {
	var nilSlice []int        // nil slice
	emptySlice := []int{}     // empty slice
	madeSlice := make([]int, 0) // empty slice

	fmt.Println(nilSlice == nil)    // true
	fmt.Println(emptySlice == nil)  // false
	fmt.Println(madeSlice == nil)   // false

	fmt.Println(len(nilSlice))      // 0
	fmt.Println(len(emptySlice))    // 0
	fmt.Println(len(madeSlice))     // 0
}
```

**Key differences**:
- A nil slice has no underlying array allocated
- An empty slice has an allocated (but empty) underlying array
- Both work identically with `append`, `len`, and `cap`
- For most use cases, prefer nil slices as they don't allocate memory

**When it matters**: JSON encoding treats them differently—nil encodes to `null`, empty slices encode to `[]`.

### Avoiding Memory Leaks with Slices

When you create a slice from a large array, the entire underlying array remains in memory even if you only reference a small portion:

```go
package main

import "fmt"

func main() {
	// Problem: memory leak
	data := make([]byte, 1000000) // 1MB array
	// ... fill data with content ...

	subset := data[0:5] // Small slice, but entire 1MB array stays in memory
	fmt.Println(len(subset))
}
```

**Solution**: Copy the data you need to a new slice to allow the large array to be garbage collected:

```go
data := make([]byte, 1000000)
// ... fill data ...

subset := make([]byte, 5)
copy(subset, data[0:5]) // Now the original 1MB array can be garbage collected
```

This is crucial in long-running applications where you extract small pieces from large datasets.

### Full Slice Expression for Capacity Control

The full slice expression `a[low:high:max]` lets you limit the capacity of the resulting slice:

```go
package main

import "fmt"

func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	// Regular slice expression
	slice1 := numbers[2:5]
	fmt.Printf("len=%d cap=%d %v\n", len(slice1), cap(slice1), slice1)
	// Output: len=3 cap=8 [2 3 4]

	// Full slice expression
	slice2 := numbers[2:5:5]
	fmt.Printf("len=%d cap=%d %v\n", len(slice2), cap(slice2), slice2)
	// Output: len=3 cap=3 [2 3 4]
}
```

**Why this matters**: Limiting capacity prevents accidental modification of elements beyond the intended slice, useful when passing slices to untrusted code or preventing aliasing bugs.

### Performance Considerations: Pre-allocation

Pre-allocating slices when you know the final size significantly improves performance:

```go
package main

import (
	"fmt"
	"time"
)

func withoutPreallocation() {
	start := time.Now()
	var numbers []int
	for i := 0; i < 1000000; i++ {
		numbers = append(numbers, i)
	}
	fmt.Println("Without preallocation:", time.Since(start))
}

func withPreallocation() {
	start := time.Now()
	numbers := make([]int, 0, 1000000)
	for i := 0; i < 1000000; i++ {
		numbers = append(numbers, i)
	}
	fmt.Println("With preallocation:", time.Since(start))
}

func main() {
	withoutPreallocation()
	withPreallocation()
}
```

**Performance impact**: Pre-allocation can be 2-10x faster for large slices because it eliminates multiple reallocation and copy operations.

**Best practice**: Use `make([]T, 0, capacity)` when you know the final size, especially for slices that will grow beyond 1000 elements.

## Index out of Range Error

In programming languages like JS/TS, if we reference an index which does not exist, we would get `undefined` value but in Go we would get an error:

```go
package main

import "fmt"

func main() {
	var numbers = []int{1, 2, 3}
	fmt.Println(numbers[3])
}
```

In the output we have:

```text
go run .
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
main.main()
	/main.go:8 +0x24
exit status 2
```

**Important distinction**:
- For **arrays** with constant indices, the Go compiler can detect out-of-bounds access at **compile-time** and your IDE will warn you
- For **slices**, or arrays with variable indices, out-of-bounds access is detected at **runtime** and causes a panic

In the above example with a slice, the error occurs at runtime as shown by the panic message

## Working with the `slices` Package

The [slices](https://pkg.go.dev/slices) package (introduced in Go 1.21) provides generic functions for common slice operations. Here are some practical examples:

### Comparing Slices

```go
package main

import (
	"fmt"
	"slices"
)

func main() {
	numbers1 := []int{1, 2, 3, 4}
	numbers2 := []int{1, 2, 3, 4}
	numbers3 := []int{1, 2, 3, 5}

	fmt.Println(slices.Equal(numbers1, numbers2)) // true
	fmt.Println(slices.Equal(numbers1, numbers3)) // false
}
```

### Checking if Slice Contains an Element

```go
numbers := []int{1, 2, 3, 4, 5}
fmt.Println(slices.Contains(numbers, 3)) // true
fmt.Println(slices.Contains(numbers, 10)) // false
```

### Sorting Slices

```go
numbers := []int{5, 2, 8, 1, 9}
slices.Sort(numbers)
fmt.Println(numbers) // [1 2 5 8 9]

names := []string{"Charlie", "Alice", "Bob"}
slices.Sort(names)
fmt.Println(names) // [Alice Bob Charlie]
```

### Finding Index of an Element

```go
numbers := []int{10, 20, 30, 40, 50}
index := slices.Index(numbers, 30)
fmt.Println(index) // 2

index = slices.Index(numbers, 100)
fmt.Println(index) // -1 (not found)
```

### Reversing a Slice

```go
numbers := []int{1, 2, 3, 4, 5}
slices.Reverse(numbers)
fmt.Println(numbers) // [5 4 3 2 1]
```

### Removing Duplicates

```go
numbers := []int{1, 2, 2, 3, 3, 3, 4, 5, 5}
numbers = slices.Compact(numbers)
fmt.Println(numbers) // [1 2 3 4 5]
```

The `slices` package provides type-safe, efficient operations that eliminate the need to write custom functions for common slice manipulations.
