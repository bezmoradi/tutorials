# Go > Pointer

Before we explore pointers and how they work, let's understand the basics of how variables are stored in memory. In Go, a statically-typed language, each variable has four main parts:

-   Name
-   Type
-   Value
-   Memory address

When we mention memory, we're talking about Random Access Memory (RAM), but we typically just call it memory. We can consider memory as a container capable of holding various data pieces. A variable, such as `name := "Go"` is one of these pieces stored in that container:

```text
+-------------------------------+
|     Holding other values      |
|                               |
|                               |
|                               |
+-------------------------------+
|     Holding other values      |
|                               |
|                               |
|                               |
+-------------------------------+
|Name            name           |
|Type            string         |
|Value           Go             |
|Memory address  0x14000010070  |
+-------------------------------+
|     Holding other values      |
|                               |
|                               |
|                               |
+-------------------------------+
```

In the example provided, the specific location or address of that particular bucket (container) in the entire memory is represented by `0x14000010070`. This address helps identify and access that precise location in memory where the variable is stored.

```go
package main

import "fmt"

func main() {
	var name string = "Go"
	fmt.Printf("value of name is %v and its type is %T", name, name)
}
```

The above program would output `value of name is Go and its type is string`. As a general rule, if a function doesn't need to mutate its input or the data you're working with is small, we would use value semantics. In Go, the ampersand symbol (`&`) is an operator which is used to get the memory address of a variable.

```go
func main() {
	var name string = "Go"
	var namePointer = &name
	fmt.Printf("value of namePointer is %v and its type is %T", namePointer, namePointer)
}
```

In the output we would see `value of namePointer is 0x1400008e020 and its type is *string`. When facing a piece of code like `&name`, we can read it like "Address of name"; in other words, pointer is a variable we do not store a value in but the address of another variable. The `&` operator is inherited from C programming language, where it has been used for the address-of operation since the 1970s.  
Technically, when you see a `*` in front of other types like `int`, `string`, `float64` etc, it tells us that it is a pointer to a type (In the above example, `*string` tells us that the related variable is a pointer to a `string`; in other words, in this example `*string` tells us that `namePointer` is a pointer (aka memory address) of a variable of type `string`). When we have a variable which is a pointer to something, by placing an asterisk in front of it (aka dereferencing), we get the value that is associated with the pointer (memory address):

```go
func main() {
	var name string = "Go"
	var namePointer = &name
	fmt.Println(*namePointer) // Go
	fmt.Println(*&name) // Go
}
```

In the above snippet, the value of `*namePointer` is the same as `*&name` just because at step one (`&name`) we are getting the memory address and at the second step (`*&name`) we are getting the value stored in that memory address. As the memory address of both `name` and `namePointer` is the same (e.g. `0x14000010070`), changing the value of `namePointer` to something else will change the value of `name` as well which is shown below:

```go
func main() {
	var name string = "Go"
	var namePointer = &name
	*namePointer = "Golang"
	fmt.Println(*namePointer) // Golang
	fmt.Println(name) // Golang
}
```

In this case, as `namePointer` is a pointer to a memory location, in order to access its value, we need to dereference it; in other words, we have to place an asterisk in front of it (`*namePointer`). From now on, we have access to the value which is stored in that memory location. As both `name` and `namePointer` refer to the same memory location, after updating the value of `namePointer` at this line:

```go
*namePointer = "Golang"
```

This change would result in also updating the value that `name` is holding because, as mentioned previously, both `name` and `namePointer` refer to the same memory location. The following diagram is showing this process visually:

```text
                                         +-------------------------------+
                                         |     Holding other values      |
                                         |                               |
                                         |                               |
                                         |                               |
                                         +-------------------------------+
                                         |     Holding other values      |
+-----------------------+                |                               |
|var name string = "Go" |-------+        |                               |
+-----------------------+       |        |                               |
                                |        +-------------------------------+
                                |        |Name            name           |
                                |        |Type            string         |
                                |        |Value           Go             |
                                +------->|Memory address  0x14000010070  |
                                |        +-------------------------------+
+-----------------------+       |        |     Holding other values      |
|var namePointer = &name|-------+        |                               |
+-----------------------+                |                               |
                                         |                               |
                                         +-------------------------------+
```

## Pointer & Value Semantics Defined

In Go, value semantics entail passing the actual data of a variable to a function or assigning it to another variable. This results in the creation of **an entirely independent copy** of the data for the new variable or function parameter.

```go
package main

import "fmt"

func updateName(nameParam string) {
	nameParam = "Golang"
	fmt.Printf("value of nameParam after update is %v\n", nameParam)
}

func main() {
	var name string = "Go"
	fmt.Println(name)
	updateName(name)
	fmt.Println(name)
}
```

In the above example, when we call the `updateName` function, the `name` argument is assigned to a totally new variable called `nameParam` which has its own memory address; in other words, it has a whole life of its own that's why in the output we would see:

```text
Go
value of nameParam after update is Golang
Go
```

It shows that the value of `name` after calling the `updateName` function has not changed. Now we are going to change our program to use pointers instead:

```go
func updateName(nameParam *string) {
	*nameParam = "Golang"
	fmt.Printf("value of nameParam after updating is %v\n", *nameParam)
}

func main() {
	var name string = "Go"
	fmt.Println(name)
	updateName(&name)
	fmt.Println(name)
}
```

In the above example, `updateName(&name)` refers to a pointer to the memory address of the `name` variable rather than the data itself and as shown in `updateName(nameParam *string)`, the input param must be a pointer to a `string` type. In the output we have:

```text
Go
value of nameParam after update is Golang
Golang
```

## How Do Stack/Heap Memories Relate to Value/Pointer Semantics?

Consider the stack as a storage area where the program manages function calls and local variables. When a function is invoked, the stack allocates space for its variables. Once the function completes, the stack automatically reclaims that space. While it's swift, it has a limited capacity in terms of size. The stack is efficient because allocation and deallocation are simple operations (just moving a stack pointer).

The heap is a larger and more flexible memory space where you can allocate memory dynamically. Data stored in the heap doesn't get automatically released and there needs to be a process to manage memory deallocation and it's usually done using garbage collection. Heap allocation is more expensive than stack allocation but provides flexibility for data that needs to outlive a function's scope.

**Important distinction**: Whether something is allocated on the stack or heap is determined by **escape analysis** performed by the Go compiler, not by whether you pass by value or by pointer. A value can escape to the heap even when passed by value, and a pointer might remain on the stack if the compiler determines it doesn't escape. The decision about stack vs heap allocation is independent from the decision about value vs pointer semantics.

Note: Slices, maps, and channels in Go are often called "reference types", but this is somewhat misleading. They are actually value types that contain pointers internally. When you pass a slice to a function, you're passing the slice header (a small struct containing a pointer to the underlying array, length, and capacity) by value, not passing a pointer to the slice.

```go
package main

func main() {
	name := "Go"
}
```

Technically, when we execute our program, a goroutine is created which gets its own memory bucket in stack memory (in a sense, you can think of a goroutine in Go as a lightweight thread).

```text
+-----------------------+
|     Stack Memory      |
|                       |
|                       |
|                       |
|                       |
|                       |
|                       |
|                       |
| +-------------------+ |
| |main()             | |
| |         +-------+ | |
| |         |name Go| | |
| |         +-------+ | |
| +-------------------+ |
+-----------------------+
```

As shown above, we have a new frame in our stack memory for the `main` function and inside that bucket our `name` variable is kept. At the moment, our active frame is the one that has to do with the `main` function. Now let's change the code as follows:

```go
func updateName(nameParam string) {
	nameParam = "Golang"
	fmt.Printf("value of nameParam after update is %v\n", nameParam)
}

func main() {
	name := "Go"
	updateName(name)
}
```

This is what happens in stack memory:

```text
+-----------------------+
|     Stack Memory      |
|                       |
|                       |
|                       |
| +-------------------+ |
| |updateName()       | |
| |    +------------+ | |
| |    |nameParam Go| | |
| |    +------------+ | |
| +-------------------+ |
|                       |
| +-------------------+ |
| |main()             | |
| |         +-------+ | |
| |         |name Go| | |
| |         +-------+ | |
| +-------------------+ |
+-----------------------+
```

Go assigns a new frame for the `updateName` function and the important thing here is that when we call `updateName`, Go will create a totally new variable called `nameParam` and assigns the value of `Go` to it. To prove that we can:

```go
func updateName(nameParam string) {
	nameParam = "Golang"
	fmt.Printf("memory address of nameParam is %v\n", &nameParam)
	fmt.Printf("value of nameParam after update is %v\n", nameParam)
}

func main() {
	name := "Go"
	fmt.Printf("memory address of name is %v\n", &name)
	updateName(name)
}
```

In the output we will see:

```text
memory address of name is 0x1400008e040
memory address of nameParam is 0x1400008e050
value of nameParam after update is Golang
```

Simply we can see that the memory address of `name` is different from the memory address of `nameParam`. With this behavior in mind, we come to realize that each frame is totally separated from other frames which results in immutability; meaning that a function cannot mutate data receiving from other functions which in most cases is a good practice. Now we are going to analyze what would happen if we send the memory address of the `name` variable to the other function (Memory address, pointer, and reference all signify the same concept in this context):

```go
func updateName(nameParam *string) {
	*nameParam = "Golang"
	fmt.Printf("memory address of nameParam is %v\n", nameParam)
}

func main() {
	name := "Go"
	fmt.Printf("memory address of name is %v\n", &name)
	updateName(&name)
}
```

Now in the output we have:

```text
memory address of name is 0x1400116010
memory address of nameParam is 0x1400116010
```

It means that inside the `updateName` function we are not creating a brand-new variable holding the value of `Go` anymore; instead we are passing the memory address of the `name` variable we created inside the `main` function.

```text
+-----------------------+
|     Stack Memory      |
|                       |
|                       |
|                       |
| +-------------------+ |
| |updateName()       | |
| |    +------------+ | |
| |    | nameParam  | | |
| |    |0x1400116010| | |
| |    +------------+ | |
| +-------------------+ |
|                       |
| +-------------------+ |
| |main()             | |
| |   +-------------+ | |
| |   |   name Go   | | |
| |   |0x1400116010 | | |
| |   +-------------+ | |
| +-------------------+ |
+-----------------------+
```

As shown above, the `updateName` function does not have its own copy of `Go` value; instead it has a reference to the memory bucket that the variable `name` is kept in. Now that we figured it out how the stack memory works, it's time to take a peek at some scenarios that we need to get help from the heap memory.

```go
package main

import "fmt"

func updateName(nameParam *string) {
	*nameParam = "Golang"
	fmt.Printf("value of nameParam after update is %v\n", nameParam)
}

func main() {
	name := "Go"
	updateName(&name)
	fmt.Printf("value of name after update is %v\n", name)
}
```

In the above program, we are passing a pointer to the `name` variable to the `updateName` function. When the Go compiler analyzes this code, it performs what's called [Escape Analysis](#what-is-escape-analysis) to determine whether variables should be allocated on the stack or the heap. In this particular example, the compiler detects that `name` is being shared across function boundaries through a pointer, which signals that it might need to outlive the `main` function's stack frame. To ensure safe access, the compiler typically allocates such variables on the heap. The general rule is: if the compiler determines that a function returns a memory address OR the function receives a memory address as its input params, it often allocates memory for those variables on the heap rather than the stack.

```text
+-----------------------+      +-----------------------+
|     Stack Memory      |      |      Heap Memory      |
|                       |      |                       |
|                       |      |                       |
|                       |      |                       |
|                       |      |                       |
|                       |      |                       |
|                       |      |                       |
|                       |      |                       |
| +-------------------+ |      |                       |
| |updateName()       | |      |                       |
| |+----------------+ | |      |                       |
| ||nameParam :=    | | |      |         +---+         |
| ||"Golang"        |-+-+------+-------->| # |         |
| |+----------------+ | |      |         +---+         |
| +-------------------+ |      |           |           |
|                       |      |           |           |
|                       |      |           |           |
| +-------------------+ |      |           |           |
| |main()             | |      |           |           |
| |   +-------------+ | |      |           |           |
| |   |updateName() | | |      |           |           |
| |   |             |<+-+------+-----------+           |
| |   +-------------+ | |      |                       |
| +-------------------+ |      |                       |
+-----------------------+      +-----------------------+
```

Previously we said that the Go compiler decides to use heap in two scenarios:

-   If a function receives a memory address as input
-   Or if a function returns a memory address

We have already seen the first scenario; to see the second condition in action we have:

```go
package main

import "fmt"

func updateName(nameParam string) *string {
	nameParam = "Golang"
	fmt.Printf("value of nameParam after update is %v\n", nameParam)

	return &nameParam
}

func main() {
	name := "Go"
	updated := updateName(name)

	fmt.Println(*updated)
}
```

We can see that the `updateName` function returns a reference type of `string`; meaning there might be other parts of the program that are interested in that value; that's why the compiler will use heap. But we are doing this in the cost of heap allocation; in other words, there needs to be an overhead for the garbage collector to clean up that bucket in the heap when the whole operations is done (we need to keep in mind that if we place lots of things in the heap, it will cause low performance just because the garbage collector constantly needs to analyze them to delete them if not used anymore).

## When Should We Use Value/Pointer Semantics?

Here are a couple of scenarios in which we can utilize pointers:

### Avoid Unnecessary Value Copies

Copying large structs or arrays can be inefficient and if you are working with such data, we can use pointer semantics to avoid the cost of copying data. By using pointers, we avoid unnecessary value copies. By default, Go creates a copy when passing values to functions:

```go
package main

import "fmt"

func printIt(input string) {
	fmt.Println(input)
}

func main() {
	name := "Go"
	printIt(name)
}
```

The visual representation of this behavior is as follows:

```text
+-----------------------+          +-------------------------------+
|     name := "Go"      |--------->|Name            name           |
+-----------------------+          |Type            string         |
                                   |Value           Go             |
                                   |Memory address  0x14000010070  |
                                   +-------------------------------+
                                   |     Holding other values      |
                                   |                               |
                                   |                               |
                                   |                               |
                                   +-------------------------------+
                                   |     Holding other values      |
                                   |                               |
                                   |                               |
                                   |                               |
                                   +-------------------------------+
                                   |Name            name           |
+-----------------------+          |Type            string         |
|     printIt(name)     |--------->|Value           Go             |
+-----------------------+          |Memory address  0x14000700001  |
                                   +-------------------------------+
                                   |     Holding other values      |
                                   |                               |
                                   |                               |
                                   |                               |
                                   +-------------------------------+
```

Technically, Go created a copy of `"Go"` string and passed that to the `printIt` function. After the function execution, that copy will be deleted automatically by the compiler. For large and complex values, this behavior may take up too much memory space though. As a workaround, by using a pointer, only one value is stored in memory and the address is passed around:

```go
func printIt(input *string) {
	fmt.Println(*input)
}

func main() {
	name := "Go"
	printIt(&name)
}
```

In above function, it receives the address of the memory bucket which holds the value of `"Go"` and can use that address to get the value.

### Directly Mutating Data

No need to mention that this feature will lead to unexpected results and is against Functional Programming principles. If a function needs to modify its receiver or an input param, we need to use pointer semantics (this is a common use case for setter methods that need to update the state of a struct). To prove all this, let's create a simple Go program:

```go
package main

import "fmt"

func updateName(nameParam *string) {
	*nameParam = "Golang"
}

func main() {
	var name string = "Go"
	updateName(&name)
	fmt.Println(name) // Golang
}
```

### Some Built-in Functions in Go Need Pointer Types as Input

If you're dealing with a library or built-in function, you might need to use pointer semantics (for example, `json.Unmarshal` function in Go standard library requires a pointer to a value to populate it with unmarshalled data) or as another example, `fmt.Scan` method uses pointers:

```go
package main

import "fmt"

func main() {
	var name string
	fmt.Scan(&name)
	fmt.Println("Your name is:", name)
}
```

Technically, `fmt.Scan` automatically dereferences the pointer and overrides the value that is stored in that address with the input entered by the user.

## A Pointer's Null Value

All values in Go have a so-called "Null Value"; the value that's set as a default if no value is assigned to a variable. For example, the null value of an `int` variable is `0`, a `float64` is `0`, and a string is `""`. For a pointer, it's `nil` which is a special value built-into Go. Basically, `nil` represents the absence of a memory address:

```go
var pointer *int
fmt.Println(pointer) // Prints <nil>
```

**Critical safety note**: Dereferencing a `nil` pointer causes a runtime panic. Always check for `nil` before dereferencing:

```go
package main

import "fmt"

func main() {
	var pointer *int

	// This will cause a panic: runtime error: invalid memory address or nil pointer dereference
	// fmt.Println(*pointer)

	// Safe approach: check before dereferencing
	if pointer != nil {
		fmt.Println(*pointer)
	} else {
		fmt.Println("pointer is nil")
	}
}
```

## Pointer Arithmetic in Go

Unlike C and C++, **Go does not support pointer arithmetic**. You cannot increment a pointer to move to the next memory location, nor can you perform mathematical operations on pointers. This design decision makes Go safer and prevents common memory-related bugs:

```go
// This is INVALID in Go (but valid in C):
// ptr++
// ptr = ptr + 1

// Go is designed to be memory-safe, and pointer arithmetic would violate this principle
```

If you need array-like behavior with pointer semantics, use slices instead, which provide safe, bounds-checked access to underlying arrays.

## What Is Escape Analysis?

The Go compiler makes use of **Escape Analysis**, an optimization technique, to better handle memory allocation and management. An object’s lifetime is examined in order to ascertain whether or not it can be safely allocated on the stack rather than the heap, which is the central idea behind escape analysis. Allocating an object on the stack is preferable to allocating it on the heap because the stack is a more efficient data structure.
The Go compiler takes care of doing the escape analysis for you automatically. If a function’s variables or data structures are being accessed outside of the function’s scope, the algorithm performing the escape analysis will flag it. A variable is said to have **escaped** when it is used outside of the function in which it was defined. In this case, the compiler will allocate it on the heap.  
When a function returns a memory address, that value is typically moved to the heap. This is because values in the heap can persist beyond the lifespan of a function.  
In Go, when a function receives a value (not a pointer), it gets its own copy of that value and that copy is placed on the stack memory. On the other hand, when a function receives a pointer or returns the address of a local variable within the scope of that function, it gives the compiler a hint that this value could be shared across multiple goroutines (threads) or could persist after the function is done and to make sure that the data will remain available, the Go compiler must allocate it on the heap.
When possible, the Go compiler will allocate variables that are local to a function in that function's stack frame. However, if the compiler cannot prove that the variable is not referenced after the function returns, the compiler must allocate the variable on the heap to avoid null pointer errors. Long story short, it's up to the compiler to decide whether to place a variable on stack or heap based on the results of escape analysis; in other words, there might be some occasions where we are passing a pointer to a function, but still that whole thing is stored on stack!

```go
package main

import "fmt"

func main() {
	name := "Go"
	fmt.Println(name)
}
```

If we run the above program by running `go run -gcflags -m main.go`, we will see:

```text
# command-line-arguments
./main.go:7:13: inlining call to fmt.Println
./main.go:7:13: ... argument does not escape
./main.go:7:14: name escapes to heap
```

Inlining means that instead of making a function call to `fmt.Println`, the compiler directly inserts the code of `fmt.Println` at the place where it's called (`fmt.Println(name)` in this case) during the compilation process. This process **reduces** the overhead of function calls and can potentially make the code faster. However, in the above case, you're seeing an additional message `... argument does not escape` which means that the argument you're passing to `fmt.Println(name)` doesn't escape to the heap. In Go, when an object doesn't escape to the heap, it's typically allocated on the stack, which is more efficient. Next we have the `name escapes to heap` optimization message which literally mean that `the name variable is shared between main() and fmt.Println() functions`. Now let's change our code as follows:

```go
func main() {
	name := "Go"
	fmt.Println(&name)
}
```

The only difference here is that instead of passing the `name` value, we are passing the address of the `name` variable. If we do `go run -gcflags -m main.go` again, we have:

```text
# command-line-arguments
./main.go:7:13: inlining call to fmt.Println
./main.go:6:2: moved to heap: name
./main.go:7:13: ... argument does not escape
```

As we are passing `&name` or in other words we are passing the memory address of the `name` variable, we get the optimization message of `moved to heap: name` meaning that the `name` variable is moved from stack to heap in order for other parts of the program to have access to it even after the execution of the function is finished.

## What Are Default Reference-types In Go?

While slices are sometimes called "reference types" in Go, this terminology can be misleading. More accurately, slices are value types that contain a pointer to an underlying array along with length and capacity information. When you pass a slice to a function, you're passing this slice header by value, but since it contains a pointer to the underlying array, modifications to the array elements affect the original slice:

```go
package main

import "fmt"

func mutateSliceOfInts(slice []int) {
	slice[0] = -1

}
func main() {

	numbers := []int{1, 2, 3}
	fmt.Println(numbers) // [1 2 3]
	mutateSliceOfInts(numbers)
	fmt.Println(numbers) // [-1 2 3]
}
```

We can see that the `numbers` slice's underlying array is mutated. This happens because the slice header (passed by value) contains a pointer to the underlying array. However, operations that would change the slice header itself (like append that causes reallocation) won't affect the original slice.

The same behavior applies to maps in Go, which are also implemented as descriptors containing pointers to the underlying hash table data structure:

```go
package main

import "fmt"

func updateMap(m map[string]int) {
	m["key1"] = -1
}

func main() {
	m := map[string]int{"key1": 1, "key2": 2, "key3": 3}
	fmt.Println(m) // map[key1:1 key2:2 key3:3]
	updateMap(m)
	fmt.Println(m) // map[key1:-1 key2:2 key3:3]
}
```

## When NOT to Use Pointers

While pointers are powerful, they're not always the right choice. Here are scenarios where you should prefer value semantics:

### Small Data Types
For primitive types and small structs (typically less than a few dozen bytes), passing by value is often more efficient. The overhead of dereferencing a pointer can exceed the cost of copying small values:

```go
// Prefer value semantics for small types
func processNumber(n int) int {
	return n * 2
}

// Avoid unnecessary pointers for primitives
// func processNumber(n *int) int {  // Overkill for an int
//     return *n * 2
// }
```

### Immutable Data
When you don't need to modify data, value semantics provide safety and clarity. They make it obvious that the function won't have side effects:

```go
type Point struct {
	X, Y float64
}

// Good: clearly read-only operation
func distance(p Point) float64 {
	return math.Sqrt(p.X*p.X + p.Y*p.Y)
}
```

### Avoid Premature Optimization
Don't use pointers just because you think they "might be faster". Use value semantics by default and only switch to pointers when:
- You have a proven performance bottleneck
- You need to mutate the data
- The data structure is genuinely large (hundreds of bytes or more)

The Go compiler is highly optimized and often makes better decisions than manual pointer optimization attempts.

### Thread Safety Concerns
Shared pointers across goroutines can lead to race conditions. If you don't need mutation, value semantics eliminate the entire class of data race bugs:

```go
// Safe: each goroutine gets its own copy
func processData(data MyStruct) {
	// No race condition possible
}

// Dangerous: shared mutable state
func processData(data *MyStruct) {
	// Requires synchronization (mutexes, channels, etc.)
}
```

## Useful Talks

-   [Golang pointers explained, once and for all](https://www.youtube.com/watch?v=sTFJtxJXkaY)
-   [Understanding Allocations: the Stack and the Heap](https://www.youtube.com/watch?v=ZMZpH4yT7M0)
