# Go > Variables & Types

## Intro

Go is a statically-typed language meaning the type of variables must be defined while defining them:

```go
var name string = "Go"
```

To create a variable, we need to use the `var` keyword followed by the name of the variable followed by the type then an equal sign and finally the value. In the above example, type `string` tells the Go compiler that this variable will always hold a string.
To define strings, we can only use double quotes or backticks (Single quotes are not allowed). Backticks can be handy if we need to add strings in multiple lines:

```go

func main() {
	var whatIsGo string = `An open-source programming language supported by Google
Easy to learn and great for teams
Built-in concurrency and a robust standard library
Large ecosystem of partners, communities, and tools`

	fmt.Println(whatIsGo)
}

```

In the output we'll have:

```text
An open-source programming language supported by Google
Easy to learn and great for teams
Built-in concurrency and a robust standard library
Large ecosystem of partners, communities, and tools
```

There is also a short form of creating variables in Go:

```go
name := "Go"
```

Keep in mind the Short Declaration Operator (`:=`) can ONLY be used inside a function and we cannot use that at the package-level scope. Technically, the `:=` operator lets the compiler figure out the type of variable based on its initial value (If we need to reassign a variable, we have to use the `=` operator):

```go
name := "Go"
name = "Golang"
```

In Go, we can declare multiple variables of **the same type** in just one line:

```go
var name, fullName string = "Go", "Golang"
```

We can also omit the type as follows:

```go
var name, fullName = "Go", "Golang"
```

If we omit type, we can store multiple values of different types:

```go
var name, version = "Go", 1.0
```

We can also go one step further and omit the `var` keyword:

```go
name, version := "Go", 1.0
```

## Constants

To define a value as a constant, we have to use the `const` keyword:

```go
const name string = "Go"
```

To make it less verbose, we can omit the type:

```go
const name = "Go"
```

A pro tip to keep in mind is that constants cannot be created by the `:=` operator. Since constants must have a known value at compile-time, the `:=` operator, which involves runtime inference, cannot be used to create constants.

## Basic Types in Go

Some most-used types in go are as follows:

-   `bool`: Either `true` or `false`
-   `string`: Either `"String ..."` (Back ticks can also be used but single quotes cannot be used)
-   `int`: Some examples are `0`, `-1`, and `100`
-   `uint`: Unsigned integer meaning non-negative numbers like `0` and `10`
-   `float64`: Some examples are `-10.01`, `0.0009`, and `10.001`
-   `[]byte`: Byte slice is a computer-friendly representation of strings. For example, the string `hi` will be shown in byte slice as `[104 105]`. To find the ascii value of different characters, visit https://www.asciitable.com

In the following example, although the variable holds the value of `1000`, as the type is `float64`, it is treated a decimal by the compiler:

```go
package main

import "fmt"

func main() {
	var n float64 = 1000
	fmt.Printf("%T", n) // float64
}
```

## Type Conversion

The process of turning one type into another is called Type Conversion:

```go
func main() {
	fmt.Println([]byte("Go")) // [71 111]
}
```

As shown above, first we list out the type we want then a pair of parenthesis and finally insert the value we need to convert. On the contrary, if we have a byte slice and need to turn it into a string, we can:

```go
func main() {
	byteSlice := []byte{71, 111}
	fmt.Println(string(byteSlice)) // Go
}
```

Something important about the conversion process is that it must be doable:

```go
something := "This is a string"
somethingElse := float32(something)
```

As a string cannot be converted into a floating point number, we get compile-time error as follows:

```text
cannot convert a (variable of type string) to type float32
```

## `Printf()`

`Printf` function formats according to a format specifier and writes to standard output:

-   `%v` gets replaced by the variable' value
-   `%T` gets replaced by the variable' type  
    As an example we have:

```go
package main

import "fmt"

func main() {
	name := "Go"
	fmt.Printf("%v %T", name, name) // Go string
}
```

In fact, `%v` and `%T` print the value and type of the `name` variable respectively.
