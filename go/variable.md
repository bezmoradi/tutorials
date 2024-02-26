# Go > Variables & Types

Go is a statically-typed language meaning the type of variables must be defined while defining them:

```go
var name string = "Go"
```

To create a variable, we need to use the `var` keyword followed by the name of the variable followed by the type then an equal sign and finally the value. In the above example, the `string` type tells the Go compiler that this variable will always hold a string.
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

Keep in mind the Short Declaration Operator (`:=`) can only be used inside a function and we cannot use that at the package-level scope. Technically, the `:=` operator lets the compiler figure out the type of variable based on its initial value (If we need to reassign a variable, we have to use the `=` operator):

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

## Constants in Go

To define a value as a constant, we have to use the `const` keyword:

```go
const name string = "Go"
```

To make it less verbose, we can omit the type:

```go
const name = "Go"
```

A pro tip to keep in mind is that constants cannot be created by the `:=` operator. Since constants must have a known value at compile-time, the `:=` operator, which involves runtime inference, cannot be used to create constants.

## An Intro to ASCII, Unicode, and UTF-8

Before digging any deeper, let's have a refresher on how computers work with 0s and 1s:

```text
+-------------+-------------+-------------------------+
|   1 Light   |     2^1     |   Represents 2 things   |
+-------------+-------------+-------------------------+
|   2 Light   |     2^2     |   Represents 4 things   |
+-------------+-------------+-------------------------+
|   3 Light   |     2^3     |   Represents 8 things   |
+-------------+-------------+-------------------------+
|   4 Light   |     2^4     |  Represents 16 things   |
+-------------+-------------+-------------------------+
|   5 Light   |     2^5     |  Represents 32 things   |
+-------------+-------------+-------------------------+
|   6 Light   |     2^6     |  Represents 64 things   |
+-------------+-------------+-------------------------+
|   7 Light   |     2^7     |  Represents 128 things  |
+-------------+-------------+-------------------------+
|   8 Light   |     2^8     |  Represents 256 things  |
+-------------+-------------+-------------------------+
```

In order to understand this concept better, let's imagine we have a light that has only two states: On & Off; in other words, this one light can represent two states: On or Off. Now imagine instead of a light, we have two lights which in this case we can have four different states as follows:

```text
On On Off Off
Off Off On On
On Off On Off
Off on Off On
```

The binary representation of the above chart is as follows:

```text
1 1 0 0
0 0 1 1
1 0 1 0
0 1 0 1
```

ASCII uses 8 bits (1 byte) or in other words eight different 0s and 1s to represent different characters which in total becomes 256 different characters (The word "bit" is combined by "bi" of "binary" and "t" of "digit"). With this in mind, there is no wonder that a bit can hold two numbers: 0 & 1. As an example, the capital "A" is represented by `01000001`.  
The problem with ASCII was that it could not cover all the characters of all the languages around the world then Unicode was created which by using hexadecimal characters, could cover way more symbols. UTF-8 is a convenient way of storing Unicode characters and it can store up to 4 bytes (32 bits) of data like emojis etc.

## Basic Types in Go

Some most-used types in go are as follows:

-   `bool`: Either `true` or `false`
-   `string`: For example `"this is a string"` (Backticks can also be used but single quotes cannot be used)
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

When it comes to numbers in Go, for more readability we can use the `_` character to separate zeros:

```go
number := 1_000_000 // or 1000000
```

The above code is completely valid in Go.  
When you create a new type using the `type` keyword, you're essentially creating an alias for an existing type:

```go
package main

import "fmt"

type customString string

func (s customString) logString() {
	fmt.Println(s)
}

func main() {
	var name customString = "Go"
	name.logString() // Go
}
```

In the above program, we have created an alias called `customString` for the `string` type.

## Type Conversion

The process of turning one type into another is called Type Conversion. Regarding the previous program, the custom type can also be used for type conversion between different types:

```go
func main() {
	var anotherName string = "Golang"
	customString(anotherName).logString()
}
```

In the above snippet, first we have converted the `anotherName` variable to `customString` then called the `logString` method on it. Now let's create an alias for the `int` type:

```go
package main

import (
	"fmt"
)

type intAlias int

func main() {
	var number intAlias = 7
	fmt.Println(number) // 7
}
```

In Go, you can convert the value of one type to another by using a type conversion expression. The syntax for a type conversion is `T(x)`, where `T` is the target type, and `x` is the value you want to convert. So, when you write `intAlias(7)`, you're using a type conversion to create a value of type `intAlias` from the int value `7`. As a more real-world example, let's consider the following program:

```go
package main

import "fmt"

type meter float64
type centimeter float64

func main() {
	lengthInMeters := meter(2.5)
	lengthInCentimeters := convertToCentimeters(lengthInMeters)
	fmt.Printf(
		"Length in Meters: %.2f\nLength in Centimeters: %.2f\n",
		lengthInMeters,
		lengthInCentimeters,
	)
}

func convertToCentimeters(meterLength meter) centimeter {
	return centimeter(meterLength * 100)
}
```

In this example, we have two type aliases, `meter` and `centimeter`, both based on the `float64` type. The `convertToCentimeters` function takes a length in meters (of type `meter`) and converts it to centimeters (of type `centimeter`). This allows for clear and expressive function signatures, and it makes the code more self-documenting. Let's see another example:

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

## An Intro to The `Printf` Function

The `Printf` function formats according to a format specifier and writes to standard output:

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

## How `Println` Function Works behind The Scenes

By visiting `https://pkg.go.dev/fmt#Println`, and clicking on the function name, we will see the source code of this function:

```go
func Println(a ...any) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```

This function received an unlimited number of inputs of `any` type then calls another one called `Fprintln` and if we take a peek at the docs for it at `https://pkg.go.dev/fmt#Fprintln`, we will see that this function comes with this signature:

```go
func Fprintln(w io.Writer, a ...any) (n int, err error)
```

As shown above, for the first input param it receives any type of `io.Writer` and if we go to `https://pkg.go.dev/os` and look for `Stdout` we have:

```go
var (
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

We see that `Stdout` is created by called `NewFile` which has this format:

```go
func NewFile(fd uintptr, name string) *File
```

It returns a value of type `File` and as `File` implements the `io.Writer` interface, it can be passed to the `Fprintln` function. To see all this in action we have:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Println("Hello World")
	fmt.Fprintln(os.Stdout, "Hello World")
}
```

Simply put, `Println` is a wrapper around `Fprintln` function. Also, to prove that we can pass our own custom type that implements the `Writer` interface we have:

```go
package main

import (
	"fmt"
)

type MyCustomStruct struct{}

func (m MyCustomStruct) Write(p []byte) (int, error) {
	fmt.Println("here", string(p))

	return 123456789, nil
}

func main() {
	m := MyCustomStruct{}
	fmt.Fprintln(m, "Hello World")
}
```

As the `Write` function needs to return an integer, we have returned an arbitrary integer but the main purpose here is to show how interfaces work.

## How to Test `Stdout`

Let's say we have a program as simple as this:

```go
package main

import (
	"fmt"
)

func prompt() {
	fmt.Print("This is written in Go")
}

func main() {
	prompt() // This is written in Go
}
```

To write a unit test for it we have:

```go
package main

import (
	"io"
	"os"
	"testing"
)

func TestPrompt(t *testing.T) {
	oldStdout := os.Stdout  // Saves the original standard output to restore later
	r, w, _ := os.Pipe()    // Creates a pipe to capture the standard output
	os.Stdout = w           // Now anything written to os.Stdout will be captured by the pipe
	prompt()                // Call the function
	_ = w.Close()           // Close the piple not to cause any memory leak
	os.Stdout = oldStdout   // Restores the original value of os.Stdout so that subsequent output goes to the actual standard output
	out, _ := io.ReadAll(r) // Reads the output from the pipe and stores it in the variable out
	if string(out) != "This is written in Go" {
		t.Errorf("expected 'This is written in Go' but got %v", string(out))
	}
}
```

### What Does `os.Pipe` Function Do?

The `Pipe` function creates a communication mechanism that allows one process to send data to another process. In the above example, it's used for capturing the output of a function, as you're doing in your test. It returns two `*os.File` values, one for the read end (`r`) and one for the write end (`w`) of the pipe.
