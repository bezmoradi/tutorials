# Go > Generics

Before we dive into the details of generics and their benefits, let's start by looking at some examples without using generics. This will help us identify the issues that can arise when generics are not employed:

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

Likewise, we can create stand-alone functions for `string` and some other types. It's crystal clear that this approach violates the DRY (Don't Repeat Yourself) principle. Instead we can use the `interface{}` (or `any`) type:

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

The problem with `interface{}` or `any` type is that they're too wide and would include anything. There might be some situations where although we need to accept different types of data, we know right off the bat what those types are. As you can see, the more types we have to support, the more code we need to write! Another problem we have in here is that even if the two input params are integers for example, as the return type is `any`, we cannot do the following arithmetic operation:

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

Then this this is how generics come to our rescue:

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

For a data type list like `int | string | float64` we can import the following package:

```text
$ go get golang.org/x/exp/constraints
```

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

I we take a peek at the definition of the `constraints.Ordered` type, we have:

```go
type Ordered interface {
	Integer | Float | ~string
}
```

`Integer` and `Float` are other interfaces including other types. `~string` means anything that have the underlying type of `string` is accepted as well:

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

We have created a custom type called `myStringType` which is an alias for `string`. Now let's create our own interface as before and in our data type list use `string` without tilde:

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

As soon as we create a new variable and assign this newly-created type to it and pass it to the `add` function, it starts complaining because our alias does not satisfy `myType` interface and we'll get the following compile-time error:

```text
myStringType does not satisfy myType (possibly missing ~ for string in myType)
```

To fix that we need to:

```go
type myType interface {
	int | ~string | float64
}
```

Basically it says include all value of type `string` and also all those values that have the underlying type of `string` as in `myStringType` case.

## How to Use Generics with Structs?
Here is how to do that:

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

## How to Implement A Mapping Function Using Generics?
The following program creates a mapping function like the `Array.prototype.map()` we have in JavaScript:

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
The problem though is that the `mappingFunction` only accepts integers and this is where generics can be applied in order to accept different values:

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
	fmt.Println(result)
}
```