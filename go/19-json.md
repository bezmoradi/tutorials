# Go > JSON

-   The built-in `encoding/json` package is Go is responsible for working with JSON data.

## How to Create JSON from Structs

-   To encode JSON data, we have to use the `Marshal` function:

```go
func Marshal(v any) ([]byte, error)
```

-   `Marshal` get input param of any type and returns a slice of `byte` or `error`:

```go
package main

import (
	"encoding/json"
	"fmt"
)

type language struct {
	Name     string
	UseCases []string
}

func main() {
	l := language{
		Name: "Go",
		UseCases: []string{
			"Cloud & Network Services",
			"Web Development",
			"Command-line Interfaces",
			"DevOps & Site Reliability",
		},
	}

	bs, err := json.Marshal(l)

	if err != nil {
		fmt.Println("Error:", err)
	}

	fmt.Printf("%T\n", bs)
	fmt.Printf("%v\n", string(bs))
}
```

-   It outputs:

```text
[]uint8
{"Name":"Go","UseCases":["Cloud \u0026 Network Services","Web Development","Command-line Interfaces","DevOps \u0026 Site Reliability"]}
```

-   The type of `bs` variable is `[]uint8` and by taking a peek at `https://go.dev/ref/spec#Types`, we will see that `uint8` is `the set of all unsigned  8-bit integers (0 to 255)`. Technically, in order to see a human-readable format of a `uint8` type, we need to cast it to the `string` type; in other words, `string(bs)` shows a string representation of any variable with the `uint8` type.

## How to Convert JSON to Struct

-   In order to convert a JSON format to a Go struct, we need to use the `Unmarshal` function:

```go
func Unmarshal(data []byte, v any) error
```

-   This function receives input data of a slice of byte (`[]byte`) as the first param and memory address of a struct as the second one. If this process results in any error, `Unmarshal` returns it.

```go
package main

import (
	"encoding/json"
	"fmt"
)

type language struct {
	Name     string
	UseCases []string
}

func main() {
	l := language{}
	s := `{"Name":"Go","UseCases":["Cloud \u0026 Network Services","Web Development","Command-line Interfaces","DevOps \u0026 Site Reliability"]}`
	bs := []byte(s)
	err := json.Unmarshal(bs, &l)

	if err != nil {
		fmt.Println("Error:", err)
	}

	fmt.Printf("%T\n", bs)
	fmt.Printf("%v\n", l)
}
```

-   It outputs:

```text
[]uint8
{Go [Cloud & Network Services Web Development Command-line Interfaces DevOps & Site Reliability]}
```

-   In the above program we are converting `s` to `[]byte` type; when we print the type of `bs` by calling `fmt.Printf("%T\n", bs)` though, we get `[]uint8`. Basically, `https://go.dev/ref/spec#Types` show us that `byte is an alias for uint8`; in other words, both `Marshal` and `Unmarshal` work with exactly the same type which is `[]uint8`
-   An important note while working with either `Marshal` or `Unmarshal` functions is that the struct fields **must** be PascalCase otherwise Go cannot figure it out.

## When to Use `Encode` And `Decode` Functions

-   So far we have seen that if we have Go struct and want to convert it to JSON, we can simply use `Marshal` function or when we want to convert a JSON object back to a Go struct, we can use the `Unmarshal` function.
-   Basically, the `Encode` and `Decode` can be used for the same purposes respectively. The difference though is that they work with stream of data.
-   For example, the `Decode` method reads JSON input from an `io.Reader` (like a file, network stream, or string reader) and directly decodes it into a Go value. It's more low-level and provides streaming JSON decoding, which can be useful when dealing with large JSON documents that can't fit entirely in memory. It's often used when you need to continuously read JSON objects from a stream without loading the entire stream into memory at once. But `Unmarshal` function, on the other hand, takes a slice of bytes containing JSON data and populates a Go value. It's more high-level and simpler to use as it operates on a byte slice (usually read from an entire file or an HTTP response) and parses the entire JSON at once. It's generally used when you have the entire JSON payload in memory and want to parse it into a Go value.

-   In order to understand the inner working of `Encode` And `Decode` functions, we need to know a little bit about the `Writer` interface in Go (`https://pkg.go.dev/io#Writer`).

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

-   As shown above, `Writer` is an interface meaning any other type with the `Write` method attached to it is going to be of type `Writer` and can be used wherever this interface is needed. For example, `json.NewEncoder` function needs a type of `Writer` as its input param (`https://pkg.go.dev/encoding/json#NewEncoder`).

```go
func NewEncoder(w io.Writer) *Encoder
```

-   For example, the type `File` from `https://pkg.go.dev/os` has the `Write` method attached to it (`https://pkg.go.dev/os#File.Write`) meaning that any variable of type `File` can be passed to the `NewEncoder` function simply because this function accepts any input params that matches with `io.Writer`. To see all this in action, we have:

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type language struct {
	Name     string
	UseCases []string
}

func main() {
	l := language{
		Name: "Go",
		UseCases: []string{
			"Cloud & Network Services",
			"Web Development",
			"Command-line Interfaces",
			"DevOps & Site Reliability",
		},
	}

	file, err := os.Create("output.json")
	if err != nil {
		fmt.Println("Error creating file:", err)

		return
	}

	defer file.Close()

	encoder := json.NewEncoder(file)
	encoder.Encode(l)
}
```

-   By the same token, the `NewDecoder` function accepts any type as input param that implements the `io.Reader` with this signature:

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

-   As the `os.Open` function return a `File` type and this type has a `Read` function (`https://pkg.go.dev/os#File.Read`) with the following structure:

```go
func (f *File) Read(b []byte) (n int, err error)
```

-   We can simply pass it to the `NewEncoder` function as shown below:

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type language struct {
	Name     string
	UseCases []string
}

func main() {
	l := language{}

	file, err := os.Open("output.json")

	if err != nil {
		fmt.Println("Error opening file:", err)

		return
	}

	defer file.Close()

	decoder := json.NewDecoder(file)
	err = decoder.Decode(&l)

	if err != nil {
		fmt.Println("Error decoding JSON:", err)
		return
	}

	fmt.Println(l)
}
```

## How `Println` Function Works behind The Scenes

-   By visiting `https://pkg.go.dev/fmt#Println`, and clicking on the function name, we will see the source code of this function:

```go
func Println(a ...any) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```

-   This function calls another one called `Fprintln` and if we take a peek at the docs for it at `https://pkg.go.dev/fmt#Fprintln`, we will see that this function comes with this signature:

```go
func Fprintln(w io.Writer, a ...any) (n int, err error)
```

-   As shown above, as its first input param it receives any type of `io.Writer` and if we go to `https://pkg.go.dev/os` and look for `Stdout` we have:

```go
var (
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

-   We see that `Stdout` is created by called `NewFile` which has this format:

```go
func NewFile(fd uintptr, name string) *File
```

-   It returns a value of type `File` and as `File` implements the `io.Writer` interface, it can be passed to the `Fprintln` function. To see all this in action we have:

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

-   Simply put, `Println` is a wrapper around `Fprintln` function. Also, to prove that we can pass our own custom type that implements the `Writer` interface we have:

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

-   As the `Write` function needs to return an integer, we have returned an arbitrary integer but the main purpose here is to show how interfaces work.
