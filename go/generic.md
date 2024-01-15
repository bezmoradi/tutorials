# Go > Generics

-   The problem with `interface{}` or `any` type is that its is too wide and would include anything. There might be some situations that although we need to accept different types of data, we know right off the bat with those types are. To understand the generics in Go, first let's see the below example:

```go
package main

import "fmt"

func main() {
	fmt.Println(add("Hi", "There"))
}

func add(a, b any) any {
	aInt, aIsInt := a.(int)
	bInt, bIsInt := b.(int)

	if aIsInt && bIsInt {
		return aInt + bInt
	}

	aString, aIsString := a.(string)
	bString, bIsString := b.(string)

	if aIsString && bIsString {
		return aString + " " + bString
	}

	return nil
}
```

-   As we can see, the more types we have to support, the more code we need to write!
-   Another problem we have in here is that Go cannot figure it out that if two of the input params are integer, for sure the result would be an integer that's why we can do an arithmetic operation on it.

```go
func main() {
	result := add(3, 4)

	fmt.Println(result + 1)
}
```

-   As shown in the above example, we get the following error:

```text
invalid operation: result + 1 (mismatched types any and int)compilerMismatchedTypes
var result any
```

-   To use generics we have:

```go
package main

import "fmt"

func main() {
	result := add(3, 4)

	fmt.Println(result + 1)
}

func add[T int | string](a, b T) T {
	return a + b
}
```

-   As Go would understand that the plus operator can be used both for integers and string, it automatically figures out the type of the `result` variable.
-   We can go one step further and extract the type as follows:

```go
type myType interface {
	int | string
}

func add[T myType](a, b T) T {
	return a + b
}
```

-   Now let's see the next version of the above program:

```go
package main

import "fmt"

func main() {
	var numThree myIntType = 3
	result := add(numThree, 4)

	fmt.Println(result + 1)
}

type myIntType int

type myType interface {
	~int | ~string
}

func add[T myType](a, b T) T {
	return a + b
}
```

-   We have created a custom type called `myIntType` which is an alias for `int`. As soon as we create a new variable and assign this newly-created type to it and pass it to the `add` function, it starts complaining because our alias does not satisfy `myType` interface; to fix that we need to:

```go
type myType interface {
	~int | string
}
```
- Basically it says include all value of type `int` and also all those values that have the underlying type of `int` as in `myIntType` case

-   Now let's make a change to break the app:

```go
func add[T int | bool](a, b T) T {
	return a + b
}
```

-   As soon as we change the list of allowed type to both `int` and `bool`, the IDE truly starts complaining that the plus operation CANNOT be used for boolean values. To fix this we have to switch back to previous solutions
-   We can still go one step further and
