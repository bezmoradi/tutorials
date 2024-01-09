# Go > Variables & Types

-   Go is a statically-types language; meaning the type of variables must be defined while writing apps:

```go
var name string = "Behzad"
```

-   To create a variable, we need to use the `var` keyword followed by the name of the variable followed by the type of the value then an equal sign and finally the value.
-   The type `string` tells the Go compiler that this variable will always hold a string
-   There is also a short form to creating variables in Go:

```go
name := "Behzad"
```

-   Keep in mind the Short Declaration Operator (`:=`) can ONLY be used inside a function and we cannot use that at the package level scope
-   `:=` operator tells the Go compiler that it is responsible for figuring out the type of variable based on its initial value. Keep in mind if we want to reassign a variable we have to only use the `=` operator:

```go
card := "HI"
card = "Hi there"
```

## Declare Multiple Variables on The Same Line

-   In Go, we can declare multiple variables of **the same type** in just one line:

```go
var num1, num2 float64 = 1000, 2000
```

-   We can also omit the type as follows:

```go
var num1, num2 = 1000, 2000
```

-   If we omit type, we can store multiple values of different types:

```go
var num1, str1 = 1000, "string"
```

-   We can also go one step further and omit the `var` keyword

```go
num1, str1 := 1000, "string"
```

## Constants in Go

-   To define a value as a constant, we have to use the `const` keyword

```go
const num1 = 1000
// We can also add type as follows
const num2 int = 1000
```

-   Constant cannot be created by `:=`

## Basic Types in Go

-   Some most-used types in go are as follows:

    -   `bool`: Either `true` or `false`
    -   `string`: Either `"String ..."` (Back ticks can also be used but single quotes cannot be used)
    -   `int`: Some examples are `0`, `-1`, and `100`
    -   `uint`: Unsigned integer meaning non-negative numbers like `0` and `10`
    -   `float64`: Some examples are `-10.01`, `0.0009`, and `10.001`
    -   `[]byte`: byte slice is a computer-friendly representation of strings. For example, the string `hi` will be shown in byte slice as `[104 105]`. To find the ascii value of different characters, visit https://www.asciitable.com

-   In the following example, although the variable holds the value of `1000`, as the type is `float64`, it is treated a decimal by Go

```go
var num float64 = 1000
```

## Type Conversion

-   The process of turning one type into another is called Type Conversion:

```go
func main() {
	fmt.Println([]byte("hi"))
}
```

-   As shown above, first we list out the type we want then a pair of parenthesis and then the value we have. On the contrary, if we have a byte slice and need to turn it into a string, we can:

```go
package main

import (
	"fmt"
)

func main() {
	byteSlice := []byte{104, 105}

	str := string(byteSlice)

	fmt.Println(str)
}
```

-   Something important about the conversion process is that it must be doable:

```go
something := "This is a string"
somethingElse := float32(something)
```

-   As a string cannot be converted into a floating point number, we get compile-time exception as

```text
cannot convert a (variable of type string) to type float32
```

## `Printf()`

-   Printf formats according to a format specifier and writes to standard output
    -   `%v` gets replaced by the variable Value
    -   `%T` gets replaced by the variable Type
    -   `%b` gets replaced by the binary value
    -   `%x` and `%X` get replaced by the hexadecimal values in lower case and upper case respectively


## Is It Possible to Export Variables from The Main Package

- In Go, the `main` package has a specific role; it's meant for executable programs. Other packages should not generally depend on the `main` package due to how the Go language is structured.
-   Variables in the `main` package are accessible only within the `main` package itself and cannot be imported by other packages. If you declare a variable with a capital letter in the `main` package, it won't be accessible in other packages that import the `main` package. If you want a variable to be accessible from other packages, you'll need to declare it in a separate package that other packages import.

```go
// ./shared/shared.go
package shared

var Name string = "Bez"
```

-   The we have another package as follows:

```go
// ./utils/utils.go
package utils

import (
	"fmt"
	"playground/shared"
)

func DoSomething() {
	fmt.Println(shared.Name)
}
```

-   And finally the `main()` function looks like this:

```go
// ./main.go
package main

import (
	"fmt"
	"playground/shared"
	"playground/utils"
)

func main() {
	utils.DoSomething()
	fmt.Println(shared.Name)
}
```

-   From now on, the `Name` variable inside the `shared` package is accessible in all other packages including the `package main`
