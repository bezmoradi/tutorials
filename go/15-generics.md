# Go > Generics

Generics, introduced in Go 1.18, allow you to write functions and data structures that work with multiple types while maintaining type safety. This tutorial explores when to use generics, how they work, and their performance implications.

## When to Use (and Not Use) Generics

### Use Generics When:

1. **You're writing identical code for multiple types**: When the only difference between implementations is the type being used
2. **Building general-purpose data structures**: Stacks, queues, trees, or custom collections that work with any type
3. **Creating reusable algorithms**: Sorting, searching, mapping, filtering operations that apply to multiple types
4. **Type safety matters**: When you need compile-time type checking instead of runtime type assertions

### Avoid Generics When:

1. **Different types need different implementations**: Use interfaces with different method implementations instead
2. **Performance is critical**: Generic code won't be faster than interface-based code and may add complexity
3. **The code is only used in one place**: The added complexity isn't worth it for single-use cases
4. **Simple interface{} suffices**: If you don't need type safety and the code is straightforward, interfaces may be simpler

**Golden Rule**: Use generics sparingly. Only add them when they provide significant benefits in maintainability and reusability. Generics add compile-time complexity and can make code harder to understand.

## The Problem Without Generics

Before diving into generics syntax, let's examine the problems they solve:

```go
package main

import "fmt"

func addIntegers(a, b int) int {
	return a + b
}

func main() {
	fmt.Println(addIntegers(3, 4)) // 7
}
```

In the above example, we have an `addIntegers` function which accepts two integers then add them up. We can do the same for adding up floats:

```go
package main

import "fmt"

func addFloats(a, b float64) float64 {
	return a + b
}

func main() {
	fmt.Println(addFloats(3.2, 3.8)) // 7
}
```

Likewise, we can create stand-alone functions for `string` and some other types. This approach clearly violates the DRY (Don't Repeat Yourself) principle. Instead we can use the `interface{}` (or `any`) type:

```go
package main

import (
	"fmt"
)

func add(a, b any) any {
	aInt, aIsInt := a.(int)
	bInt, bIsInt := b.(int)
	if aIsInt && bIsInt {
		return aInt + bInt
	}

	aFloat, aIsFloat := a.(float64)
	bFloat, bIsFloat := b.(float64)
	if aIsFloat && bIsFloat {
		return aFloat + bFloat
	}

	aString, aIsString := a.(string)
	bString, bIsString := b.(string)
	if aIsString && bIsString {
		return aString + " " + bString
	}

	return nil
}

func main() {
	fmt.Println(add("hi", "there")) // hi there
}
```

The problem with `interface{}` or `any` type is that they're too wide and would include anything. In many situations, we need to accept different types of data, but we know exactly which types should be allowed. As you can see, the more types we have to support, the more code we need to write. Another problem is that even when both input parameters are integers, since the return type is `any`, we cannot perform arithmetic operations on the result:

```go
func main() {
	result := add(3, 4)
	fmt.Println(result + 1)
}
```

We'll get the following error:

```text
invalid operation: result + 1 (mismatched types any and int)compilerMismatchedTypes
var result any
```

## Basic Generics Syntax

This is how generics solve these problems:

```go
package main

import (
	"fmt"
)

func add[T int | float64](a, b T) T {
	return a + b
}

func main() {
	result := add(3, 3)
	fmt.Println(result + 1) // 7
}
```

As Go would understand that the plus operator can be used both for integers and floats, it automatically figures out the type of the `result` variable. We can also go one step further and extract the type as follows:

```go
type myType interface {
	int | string | float64
}

func add[T myType](a, b T) T {
	return a + b
}
```

## Using the Constraints Package

For common type constraints like `int | string | float64`, we can use the `constraints` package:

```text
$ go get golang.org/x/exp/constraints
```

**Important Note**: The `constraints` package is experimental and located in `golang.org/x/exp`, not in the Go standard library. Its API is not covered by the Go 1 compatibility guarantee and may change in future versions. As of Go 1.26 (2026), it remains experimental.

Then refactor our code as follows:

```go
package main

import (
	"fmt"

	"golang.org/x/exp/constraints"
)

func add[T constraints.Ordered](a, b T) T {
	return a + b
}

func main() {
	fmt.Println(add(3, 4)) // 7
}
```

## Understanding Type Constraints and the ~ Operator

If we examine the definition of the `constraints.Ordered` type, we see:

```go
type Ordered interface {
	Integer | Float | ~string
}
```

`Integer` and `Float` are themselves constraint interfaces containing various numeric types.

### The Tilde (~) Operator

The `~` operator is crucial for understanding type constraints. It specifies that not only the exact type is accepted, but also any type with that underlying type.

- `string` means only the exact built-in string type
- `~string` means the built-in string type AND any custom type defined as `type MyString string`

This distinction matters when you create type aliases. Here's an example:

```go
package main

import (
	"fmt"

	"golang.org/x/exp/constraints"
)

type myStringType string

func add[T constraints.Ordered](a, b T) T {
	return a + b
}

func main() {
	var greeting myStringType = "Hello"
	fmt.Println(add(greeting, " World"))
}
```

In this example, `myStringType` is a custom type with `string` as its underlying type. Because `constraints.Ordered` uses `~string`, it accepts both regular strings and our custom type.

Now let's see what happens when we define our own constraint without the tilde:

```go
package main

import (
	"fmt"
)

type myStringType string
type myType interface {
	int | string | float64
}

func add[T myType](a, b T) T {
	return a + b
}

func main() {
	var greeting myStringType = "Hello"
	fmt.Println(add(greeting, " World"))
}
```

This code fails to compile. When we pass `greeting` (type `myStringType`) to `add`, we get this error:

```text
myStringType does not satisfy myType (possibly missing ~ for string in myType)
```

The error message is helpful—it suggests adding `~` before `string`. The problem is that our constraint only accepts the exact `string` type, not types that have `string` as their underlying type.

To fix this, add the tilde operator:

```go
type myType interface {
	int | ~string | float64
}
```

**Key Insight**: Without `~`, the constraint accepts only `string`. With `~string`, it accepts both `string` and any type like `myStringType` where the underlying type is `string`. This makes generic code more flexible and reusable when working with custom types.

## Performance Considerations

Before adopting generics, understand their performance characteristics:

### Compile-Time vs Runtime Costs

**Generics work via monomorphization**: The Go compiler generates separate versions of generic functions for each type used. This happens at compile time.

**Implications**:
- **Compile time increases**: More type instantiations mean longer compilation
- **Binary size grows**: Each type instantiation adds code to the binary
- **No runtime overhead**: Unlike interfaces, no dynamic dispatch or boxing occurs

### Generics vs Interfaces

**Common misconception**: Generics are faster than interfaces.

**Reality**: For most use cases, performance is comparable:
- Generics eliminate interface boxing/unboxing overhead
- But interfaces enable runtime polymorphism that generics cannot
- Hot paths benefit more from generics; cold paths won't see meaningful differences

**Example**: If you're calling a function millions of times in a tight loop, generics may help. For occasional calls, the complexity isn't worth it.

### When Performance Matters

Use generics for performance when:
1. The function is in a critical hot path (profiling confirms this)
2. You're eliminating repeated type assertions
3. You're building performance-critical data structures (custom collections)

**Don't use generics for performance** when:
- You haven't profiled and confirmed a bottleneck
- The code is not in a hot path
- The added complexity outweighs marginal gains

**Best Practice**: Profile first, optimize second. Don't assume generics will make code faster—measure it.

## How to Use Generics with Structs

Here is how to add generic type parameters to structs:

```go
package main

import "fmt"

type user[T int | string] struct {
	name     string
	age      int
	metadata T
}

func main() {
	u := user[int]{name: "John", age: 40, metadata: 100}
	fmt.Println(u) // {John 40 100}
}
```

## Implementing a Generic Mapping Function

A common use case for generics is implementing utility functions like map, filter, and reduce. Let's build a mapping function similar to JavaScript's `Array.prototype.map()`.

### Same-Type Transformations

First, a simple version without generics:

```go
package main

import "fmt"

func mappingFunction(input []int, callback func(int) int) []int {
	var total []int
	for _, v := range input {
		total = append(total, callback(v))
	}
	return total
}

func double(input int) int {
	return input * 2
}

func main() {
	result := mappingFunction([]int{1, 2, 3}, double)
	fmt.Println(result) // [2 4 6]
}
```

The problem is that `mappingFunction` only works with integers. Let's make it generic:

```go
package main

import "fmt"

type numbers interface {
	int | float64
}

func mappingFunction[T numbers](input []T, callback func(T) T) []T {
	var total []T
	for _, v := range input {
		total = append(total, callback(v))
	}
	return total
}

func double[T numbers](input T) T {
	return input * 2
}

func main() {
	result := mappingFunction([]float64{1.1, 2.1, 3.1}, double)
	fmt.Println(result) // [2.2 4.2 6.2]
}
```

### Different-Type Transformations

The previous example transforms `[]T` to `[]T` (same type). Often we need to transform from one type to another. For example, converting integers to strings:

```go
package main

import (
	"fmt"
	"strconv"
)

// Map transforms a slice of type T to a slice of type U
func Map[T any, U any](input []T, callback func(T) U) []U {
	result := make([]U, len(input))
	for i, v := range input {
		result[i] = callback(v)
	}
	return result
}

func main() {
	// Transform []int to []string
	numbers := []int{1, 2, 3, 4, 5}
	strings := Map(numbers, func(n int) string {
		return strconv.Itoa(n)
	})
	fmt.Println(strings) // [1 2 3 4 5]

	// Transform []string to []int (get lengths)
	words := []string{"hello", "world", "go"}
	lengths := Map(words, func(s string) int {
		return len(s)
	})
	fmt.Println(lengths) // [5 5 2]

	// Transform []int to []bool (check if even)
	nums := []int{1, 2, 3, 4, 5}
	evenFlags := Map(nums, func(n int) bool {
		return n%2 == 0
	})
	fmt.Println(evenFlags) // [false true false true false]
}
```

This demonstrates generics' real power: creating reusable utilities that work across arbitrary type combinations while maintaining type safety. The compiler ensures the callback's input type matches the slice element type, and the output type matches the result slice.


## Generic Types with Structs

In Go, we can add generics to functions; likewise, we can add generic types to structs as well:

```go
package main

import "fmt"

type person[T string | int] struct {
	lastName  string
	firstName string
	metadata  T
}

func main() {
	p := person[string]{
		firstName: "John",
		lastName:  "Doe",
		metadata:  "this is a generic type",
	}
	fmt.Println(p)
}
```

As shown above, the type of `metadata` can be either `string` or `int`.

## Advanced Concepts

### Multiple Type Parameters

Functions and types can have multiple type parameters:

```go
package main

import "fmt"

// Pair holds two values of potentially different types
type Pair[T any, U any] struct {
	First  T
	Second U
}

// MakePair creates a pair with type inference
func MakePair[T any, U any](first T, second U) Pair[T, U] {
	return Pair[T, U]{First: first, Second: second}
}

func main() {
	// String and int
	p1 := MakePair("age", 30)
	fmt.Printf("%s: %d\n", p1.First, p1.Second) // age: 30

	// Two different numeric types
	p2 := MakePair(3.14, 42)
	fmt.Printf("Pi: %v, Answer: %v\n", p2.First, p2.Second) // Pi: 3.14, Answer: 42
}
```

### Generic Type Aliases (Go 1.24+)

Since Go 1.24, type aliases can be parameterized:

```go
package main

import "fmt"

// Generic type alias
type StringMap[T any] = map[string]T

// Another generic alias
type Pair[T any] = [2]T

func main() {
	// Using StringMap alias
	users := StringMap[int]{
		"alice": 30,
		"bob":   25,
	}
	fmt.Println(users) // map[alice:30 bob:25]

	// Using Pair alias
	coords := Pair[float64]{3.14, 2.71}
	fmt.Println(coords) // [3.14 2.71]
}
```

### Method Sets and Type Parameters

Generic types can have methods, and you can constrain type parameters by their method sets:

```go
package main

import "fmt"

// Stringer interface
type Stringer interface {
	String() string
}

// Container that only accepts types implementing Stringer
type Container[T Stringer] struct {
	items []T
}

func (c *Container[T]) Add(item T) {
	c.items = append(c.items, item)
}

func (c *Container[T]) PrintAll() {
	for _, item := range c.items {
		fmt.Println(item.String())
	}
}

// Custom type implementing Stringer
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

func main() {
	container := Container[Person]{}
	container.Add(Person{Name: "Alice", Age: 30})
	container.Add(Person{Name: "Bob", Age: 25})
	container.PrintAll()
	// Output:
	// Alice (30 years old)
	// Bob (25 years old)
}
```

### Self-Referential Type Constraints (Go 1.26+)

Starting with Go 1.26, generic types can reference themselves in their type parameter constraints. This enables powerful patterns like comparable nodes in trees:

```go
package main

import "fmt"

// Comparable defines types that can be compared
type Comparable[T any] interface {
	CompareTo(T) int
}

// Node that can compare itself to other nodes of the same type
type Node[T Comparable[T]] struct {
	Value T
	Left  *Node[T]
	Right *Node[T]
}

// Custom type implementing Comparable
type Integer int

func (i Integer) CompareTo(other Integer) int {
	if i < other {
		return -1
	} else if i > other {
		return 1
	}
	return 0
}

func main() {
	node := Node[Integer]{
		Value: Integer(10),
	}
	fmt.Println(node.Value) // 10
}
```

### Limitations and Edge Cases

**Operator Constraints**: Not all operators work with all types. For example:

```go
// This works for int and float64 (they support *)
func multiply[T int | float64](a, b T) T {
	return a * b
}

// This won't work - can't multiply strings
// func multiply[T int | string](a, b T) T {
// 	return a * b // Compile error: invalid operation
// }
```

**Type Inference**: Sometimes Go can't infer types and you must specify them explicitly:

```go
func main() {
	// Type inference works here
	result := Map([]int{1, 2, 3}, strconv.Itoa)

	// Might need explicit types in complex cases
	result2 := Map[int, string]([]int{1, 2, 3}, strconv.Itoa)
}
```

**Cannot use type parameters in const declarations**:

```go
// This doesn't work
// func example[T int | float64]() {
// 	const max T = 100 // Compile error
// }
```

These advanced features enable sophisticated abstractions while maintaining Go's emphasis on clarity and simplicity. Use them when they solve real problems, not for the sake of complexity.
