# Go > Variables & Types

Go is a statically-typed language, meaning variable types are determined at compile-time rather than runtime:

```go
var name string = "Go"
```

To create a variable, we need to use the `var` keyword followed by the name of the variable followed by the type then an equal sign and finally the value. In the above example, the `string` type tells the compiler that this variable will **always** store a string.
To define strings, we can only use double quotes or backticks (single quotes are not allowed). Backticks can be handy if we need to add strings in multiple lines:

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

Keep in mind the Short Declaration Operator (`:=`) can only be used inside a function and we cannot use that at the package-level scope. Technically, the `:=` operator lets the compiler figure out the type of variable based on its initial value (if we need to reassign a variable, we have to use the `=` operator):

```go
name := "Go"
name = "Golang"
```

We can also append the previous value to the new one by using the `+=` operation as follows:

```go
name := "Go"
name += "lang"
```

In this case, the new value of the `name` variable will be "Golang". In Go, we can declare multiple variables of **the same type** in just one line:

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

A pro tip to keep in mind is that constants cannot be created by the `:=` operator. The `:=` syntax is specifically the short variable declaration operator, and by definition it declares variables, not constants. Constants must be declared using the `const` keyword.

## Zero Values

In Go, variables declared without an explicit initial value are automatically given their **zero value**. This is different from many languages where uninitialized variables contain garbage values or cause errors. Go's zero values are:

```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string

	fmt.Printf("%v %v %v %q\n", i, f, b, s) // 0 0 false ""
}
```

The zero values for common types are:
- `0` for numeric types (`int`, `float64`, etc.)
- `false` for boolean type
- `""` (empty string) for strings
- `nil` for pointers, slices, maps, channels, functions, and interfaces

This design decision makes Go programs more predictable and eliminates a whole class of bugs related to uninitialized variables. It also means you can often skip initialization if the zero value is what you need.

**Note for custom types:** Defined types inherit the zero value of their underlying type:

```go
type UserID int    // zero value is 0
type Status string // zero value is ""
```

## An Intro to ASCII, Unicode, and UTF-8

Before digging any deeper, let's have a refresher on how computers work with 0s and 1s:

```text
+-------------+-------------+-------------------------+
|   1 Light   |     2^1     |   Represents 2 States   |
+-------------+-------------+-------------------------+
|   2 Lights  |     2^2     |   Represents 4 States   |
+-------------+-------------+-------------------------+
|   3 Lights  |     2^3     |   Represents 8 States   |
+-------------+-------------+-------------------------+
|   4 Lights  |     2^4     |  Represents 16 States   |
+-------------+-------------+-------------------------+
|   5 Lights  |     2^5     |  Represents 32 States   |
+-------------+-------------+-------------------------+
|   6 Lights  |     2^6     |  Represents 64 States   |
+-------------+-------------+-------------------------+
|   7 Lights  |     2^7     |  Represents 128 States  |
+-------------+-------------+-------------------------+
|   8 Lights  |     2^8     |  Represents 256 States  |
+-------------+-------------+-------------------------+
```

To understand this concept better, imagine a light with two states: On or Off. With two lights, we can represent four different states:

```text
OFF OFF → 0 0
OFF ON  → 0 1
ON OFF  → 1 0
ON ON   → 1 1
```

ASCII uses 8 bits (1 byte) or in other words eight different 0s and 1s to represent different characters which in total becomes 256 different characters (the word "bit" is combined by "bi" of "binary" and "t" of "digit"). A single bit can hold one value that can be either 0 or 1. As an example, the capital "A" is represented by `01000001`.

The problem with ASCII was that it could not cover all the characters of all the languages around the world. Unicode was created to solve this by defining a much larger set of code points - over 1.1 million possible characters instead of ASCII's 256. Each character is assigned a unique number (code point), often written in hexadecimal notation like U+0041 for "A".

UTF-8 is an efficient encoding scheme for storing Unicode characters. It uses 1-4 bytes per character: ASCII characters use just 1 byte (backward compatible), while characters from other languages and emojis use 2-4 bytes.

## Basic Types in Go

Some most-used types in Go are as follows:

-   `bool`: Either `true` or `false`
-   `string`: For example `"this is a string"` (backticks can also be used but single quotes cannot be used)
-   `int`: Platform-dependent signed integer (32 or 64 bits depending on the system). Examples: `0`, `-1`, `100`
-   `int8`, `int16`, `int32`, `int64`: Fixed-size signed integers (8, 16, 32, or 64 bits)
-   `uint`: Platform-dependent unsigned integer (32 or 64 bits)
-   `uint8`, `uint16`, `uint32`, `uint64`: Fixed-size unsigned integers
-   `byte`: Alias for `uint8`, commonly used for raw data
-   `rune`: Alias for `int32`, represents a Unicode code point. Used when working with individual characters
-   `float32`, `float64`: Floating-point numbers. Examples: `-10.01`, `0.0009`, `10.001`
-   `complex64`, `complex128`: Complex numbers with float32 or float64 parts
-   `[]byte`: Byte slice is a computer-friendly representation of strings. For example, the string `hi` will be shown in byte slice as `[104 105]`. To find the ASCII value of different characters, visit [ASCII Table](https://www.asciitable.com)

**Important distinctions:**
- `int` vs `int64`: Even on a 64-bit system, `int` and `int64` are different types and require explicit conversion
- `rune` vs `byte`: When iterating over strings, use `rune` to handle Unicode characters properly (multi-byte characters like emojis)
- **When to use platform-dependent vs sized types:**
  - Use `int` for: array indices, loop counters, lengths, general arithmetic (it's optimized for the platform)
  - Use sized types (`int32`, `int64`, etc.) for: binary protocols, file formats, serialization, explicit bit manipulation, or when you need guaranteed size across platforms
  - Prefer `float64` over `float32` unless memory is constrained (better precision, and most math functions use `float64` anyway)

In the following example, although the variable holds the value of `1000`, as the type is `float64`, it is treated as a decimal by the compiler:

```go
package main

import "fmt"

func main() {
	var n float64 = 1000
	fmt.Printf("%T", n) // float64
}
```

When it comes to numbers in Go, for more readability we can use the `_` character as a digit separator:

```go
number := 1_000_000 // Equals 1000000
```

### Working with Runes and Unicode

Since Go strings are UTF-8 encoded, a single character might be multiple bytes. The `rune` type (alias for `int32`) represents a Unicode code point and is essential for correctly handling international text:

```go
package main

import "fmt"

func main() {
	s := "Hello 世界"

	// Using bytes - WRONG for Unicode
	fmt.Println(len(s)) // 13 (not 8!)

	// Using runes - CORRECT
	for i, r := range s {
		fmt.Printf("Index %d: %c (Unicode: U+%04X)\n", i, r, r)
	}

	// Convert string to rune slice
	runes := []rune(s)
	fmt.Println(len(runes)) // 8 (actual character count)
}
```

The key takeaway: use `[]byte` for raw data, use `[]rune` when you need to count or manipulate actual characters.

## Custom Types (Defined Types)

When you create a new type using the `type` keyword, you're creating a **defined type** (also called a named type) based on an existing type. This is different from a type alias and creates a distinct type that is not interchangeable with its underlying type:

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

In the above program, we have created a new defined type called `customString` based on the `string` type. As the `logString` method is attached to the `customString` type, any variable of this type has access to that method. The key concept here is that in Go, you can take existing types (like `string`) and create your own distinct version, then add special behaviors (methods) to them. So instead of just having a regular `string`, you now have a `customString` that knows how to print itself using the `logString()` method.

Note that `customString` and `string` are **not interchangeable** without explicit conversion, even though they have the same underlying structure. This type safety prevents bugs. If you want a true alias that IS interchangeable, use `type customString = string` (with `=`).

## Type Conversion

The process of turning one type into another is called Type Conversion. Regarding the previous program, the custom type can also be used for type conversion between different types:

```go
package main

import "fmt"

type customString string

func (s customString) logString() {
	fmt.Println(s)
}

func main() {
	var anotherName string = "Golang"
	customString(anotherName).logString()
}
```

In the above snippet, first we have converted the `anotherName` variable to `customString` then called the `logString` method on it.  
In Go, you can convert the value of one type to another by using a type conversion expression. The syntax for a type conversion is `T(x)`, where `T` is the target type, and `x` is the value you want to convert.

Imagine an API that accepts timeouts. One function expects seconds, another expects milliseconds. Using plain `float64` or `int` makes it very easy to pass the wrong unit. In real systems, developers constantly deal with milliseconds, seconds, and minutes and bugs happen when these get mixed up:

```go
package main

import "fmt"

type Second float64
type Millisecond float64

func (s Second) ToMilliseconds() Millisecond {
	return Millisecond(s * 1000)
}

func (s Second) IsTooLong() bool {
	return s > 30
}

func main() {
	requestTimeout := Second(2.5)

	fmt.Println(requestTimeout.ToMilliseconds()) // 2500
	fmt.Println(requestTimeout.IsTooLong())      // false
}
```

This is better than primitive types simply because you cannot accidentally pass Milliseconds where Seconds is expected. The compiler enforces type safety at compile-time with zero runtime overhead.

**Why this matters in production:** This pattern is used extensively in real-world Go code. For example, the standard library's `time.Duration` is a defined type based on `int64`. This prevents errors like adding a duration to a timestamp, or confusing nanoseconds with milliseconds. The type system catches these errors at compile-time rather than in production.

**Performance note:** Custom types have no runtime cost. The conversion `Second(2.5)` is purely a compile-time construct that tells the type checker "treat this `float64` as a `Second`". At runtime, it's still just a `float64` in memory.

Let's see another example:

```go
package main

import "fmt"

func main() {
	fmt.Println([]byte("Go")) // [71 111]
}
```

As shown above, first we list out the type we want then a pair of parentheses and finally insert the value we need to convert. On the contrary, if we have a byte slice and need to turn it into a string, we can:

```go
package main

import "fmt"

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

**Type conversion gotchas for staff engineers:**

```go
package main

import "fmt"

func main() {
	// Narrowing conversions can lose data
	var bigInt int64 = 9223372036854775807 // max int64
	var smallInt int8 = int8(bigInt)       // truncates to -1
	fmt.Println(smallInt)                  // -1

	// Float to int truncates, doesn't round
	var f float64 = 3.99
	var i int = int(f)
	fmt.Println(i) // 3, not 4

	// int to float64 is usually safe, but not for very large ints
	var largeInt int64 = 9007199254740993
	var asFloat float64 = float64(largeInt)
	fmt.Println(int64(asFloat) == largeInt) // false - precision lost
}
```

Go type conversions are **explicit and unchecked** - the compiler won't warn you about data loss. In production code, validate ranges before converting or use libraries that provide checked conversions.

## An Intro to the `Printf` Function

The `Printf` function formats according to a format specifier and writes to standard output:

-   `%v` gets replaced by the variable's value
-   `%T` gets replaced by the variable's type

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

Other useful format specifiers include:
- `%d` for decimal integers
- `%f` for floating-point numbers
- `%s` for strings
- `%t` for booleans
- `%+v` for struct values with field names

```go
package main

import "fmt"

func main() {
	age := 25
	price := 19.99
	active := true

	fmt.Printf("Age: %d, Price: $%.2f, Active: %t\n", age, price, active)
	// Output: Age: 25, Price: $19.99, Active: true
}
```

The `Printf` function is essential for formatted output and debugging in Go programs.
