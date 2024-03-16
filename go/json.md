# Go > JSON

The built-in `encoding/json` package is Go is responsible for working with JSON data. To encode JSON data, we have to use the `Marshal` function which has the following signature:

```go
func Marshal(v any) ([]byte, error)
```

As shown above, `Marshal` gets input param of any type and returns`[]byte` and `error` (Read `[]byte` as "a slice of byte"). To make our code testable, we are goring to create a function called `jsonEncoder` as follows:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type language struct {
	Name     string   `json:"name,omitempty"`
	UseCases []string `json:"use_cases,omitempty"`
}

func jsonEncoder(input any) (string, error) {
	byteSlice, err := json.Marshal(input)
	if err != nil {
		return "", err
	}

	return string(byteSlice), nil
}

func main() {
	l := language{
		Name: "Go",
		UseCases: []string{
			"Cloud And Network Services",
			"Web Development",
			"Command-line Interfaces",
			"DevOps And Site Reliability",
		},
	}
	j, err := jsonEncoder(l)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(j)
}
```

It outputs:

```text
{"name":"Go","use_cases":["Cloud And Network Services","Web Development","Command-line Interfaces","DevOps And Site Reliability"]}
```

An important note while working with either `Marshal` or `Unmarshal` functions is that the initial letter of struct fields must be uppercase otherwise Go cannot figure it out.  
The type of `byteSlice` variable is `[]byte` and by taking a peek at `https://go.dev/ref/spec#Types`, we will see that it's an alias for `uint8` which is `the set of all unsigned 8-bit integers (0 to 255)`. Technically, in order to see a human-readable format of a `uint8` type (aka `byte`), we need to cast it to the `string` type; in other words, `string(byteSlice)` shows a string representation of any variable with the `uint8` type. Now let's create a file named `main_test.go` to add tests for our function:

```go
package main

import (
	"testing"
)

type Node struct {
	Value string
	Next  *Node
}

func TestJsonEncoder(t *testing.T) {
	node1 := Node{Value: "Node 1", Next: nil}
	node2 := Node{Value: "Node 2", Next: nil}
	node1.Next = &node2
	node2.Next = &node1

	tests := []struct {
		name         string
		input        any
		output       string
		errorMessage string
	}{
		{
			name:   "valid input",
			input:  language{Name: "Go", UseCases: []string{"Cloud"}},
			output: `{"name":"Go","use_cases":["Cloud"]}`,
		},
		{
			name:   "valid input with some empty fields",
			input:  language{Name: "", UseCases: []string{}},
			output: `{}`,
		},
		{
			name:         "invalid syclic data structure input",
			input:        node1,
			errorMessage: "json: unsupported value: encountered a cycle via *main.Node",
		},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			result, err := jsonEncoder(test.input)
			if err != nil && test.errorMessage != err.Error() {
				t.Errorf("expected %v but received %v", test.errorMessage, err.Error())
			}
			if result != test.output {
				t.Errorf("expected %v, but received %v", test.output, result)
			}
		})
	}
}
```

This is a table test to test different edge cases.

## How to Convert JSON to Struct

In order to convert a JSON format to a struct, we need to use the `Unmarshal` function:

```go
func Unmarshal(data []byte, v any) error
```

This function receives input data of a slice of bytes (`[]byte`) as the first param and memory address of a struct as the second one. If this process results in any error, `Unmarshal` returns it. In the following snippet, we have created a function called `jsonDecoder` which utilized the `Unmarshal` function:

```go
package main

import (
	"encoding/json"
	"fmt"
)

type language struct {
	Name     string   `json:"name,omitempty"`
	UseCases []string `json:"use_cases,omitempty"`
}

func jsonDecoder(input string) (language, error) {
	l := language{}
	err := json.Unmarshal([]byte(input), &l)
	if err != nil {
		return language{}, err
	}

	return l, nil
}

func main() {
	input := `{"name":"Go","use_cases":["Cloud And Network Services","Web Development","Command-line Interfaces","DevOps And Site Reliability"]}`
	language, err := jsonDecoder(input)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println(language)
}
```

It outputs:

```text
{Go [Cloud And Network Services Web Development Command-line Interfaces DevOps And Site Reliability]}
```

In the above program we are converting the `input` param to `[]byte` type. If we print the type of `[]byte(input)` by calling `fmt.Printf("%T\n", []byte(input))`, we get `[]uint8`. Basically, `https://go.dev/ref/spec#Types` shows us that `byte is an alias for uint8`; in other words, both `Marshal` and `Unmarshal` work with exactly the same type which is `[]uint8`. Now let's add some tests to check the correctness of the `jsonDecoder` function:

```go
package main

import (
	"reflect"
	"testing"
)

func TestJsonDecoder(t *testing.T) {
	tests := []struct {
		name         string
		input        string
		output       language
		errorMessage string
	}{
		{
			name:   "valid json input",
			input:  `{"name":"Go","use_cases":["Cloud"]}`,
			output: language{Name: "Go", UseCases: []string{"Cloud"}},
		},
		{
			name:   "valid json input with missing field",
			input:  `{"name":"Go"}`,
			output: language{Name: "Go"},
		},
		{
			name:         "badly formatted json",
			input:        `{"name":"Go}`,
			errorMessage: "unexpected end of JSON input",
		},
		{
			name:         "non-json input",
			input:        `this is not json`,
			errorMessage: "invalid character 'h' in literal true (expecting 'r')",
		},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			result, err := jsonDecoder(test.input)
			if err != nil && test.errorMessage != err.Error() {
				t.Fatalf("expected %v but received %v", test.errorMessage, err.Error())
			}
			if !reflect.DeepEqual(result, test.output) {
				t.Errorf("expected %v, but received %v", test.output, result)
			}
		})
	}
}
```

## When to Use `Encode` And `Decode` Functions

So far we have seen that if we have a struct and want to convert it to JSON, we can simply use `Marshal` function or when we want to convert a JSON object back to a struct, we can use the `Unmarshal` function. Basically, the `Encode` and `Decode` can be also used for the same purposes respectively. The difference though is that they work with a stream of data.  
For example, the `Decode` method reads JSON input from an `io.Reader` (like a file, network stream, or string reader) and directly decodes it into a value. It's more low-level and provides streaming JSON decoding which can be useful when dealing with large JSON documents that can't fit entirely in memory. It's often used when you need to continuously read JSON objects from a stream without loading the entire stream into memory at once. But `Unmarshal` function, on the other hand, takes a slice of bytes containing JSON data and populates a struct. It's more high-level and simpler to use as it operates on a byte slice (usually read from an entire file or an HTTP response) and parses the entire JSON at once. It's generally used when you have the entire JSON payload in memory and want to parse it.
In order to understand the inner working of `Encode` And `Decode` functions, we need to know a little bit about the `Writer` interface in Go (`https://pkg.go.dev/io#Writer`).

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

As shown above, `Writer` is an interface meaning any other type with the `Write` method attached to it is going to be of type `Writer` and can be used wherever this interface is needed. For example, [json.NewEncoder](https://pkg.go.dev/encoding/json#NewEncoder) function needs a type of `Writer` as its input param:

```go
func NewEncoder(w io.Writer) *Encoder
```

For example, the [File](https://pkg.go.dev/os) type has the [Write](https://pkg.go.dev/os#File.Write) method attached to it meaning any variable of type `File` can be passed to the `NewEncoder` function simply because this function accepts any input params that matches with `io.Writer`. To see all this in action, we have:

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

By the same token, the `NewDecoder` function accepts any type as input param that implements the `io.Reader` with this signature:

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

As the `os.Open` function returns a `File` type and this type has a [Read](https://pkg.go.dev/os#File.Read) function with the following signature:

```go
func (f *File) Read(b []byte) (n int, err error)
```

We can simply pass it to the `NewEncoder` function as shown below:

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
