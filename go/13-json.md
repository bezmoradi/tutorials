# Go > JSON

## Why JSON Encoding and Decoding Matter

JSON (JavaScript Object Notation) is the de facto standard for data interchange in modern applications. Whether you're building REST APIs, consuming third-party services, or storing configuration data, you'll need to convert between Go's strongly-typed structs and JSON's text-based format.

Go's static type system means this conversion must be explicit. Unlike dynamically-typed languages where JSON maps naturally to native data structures, Go requires deliberate marshaling (Go → JSON) and unmarshaling (JSON → Go). The `encoding/json` package in the standard library provides these capabilities with zero external dependencies.

## Converting Structs to JSON with Marshal

The built-in `encoding/json` package in Go is responsible for working with JSON data. To encode JSON data, we use the `Marshal` function which has the following signature:

```go
func Marshal(v any) ([]byte, error)
```

As shown above, `Marshal` accepts a parameter of any type and returns `[]byte` and `error` (read `[]byte` as "a slice of bytes").

**Why []byte instead of string?** Strings in Go are immutable. Using mutable byte slices (`[]byte`) allows the JSON encoder to build the output efficiently without creating multiple intermediate string copies. Similarly, parsing JSON benefits from working with mutable buffers.

To make our code testable, we are going to create a function called `jsonEncoder` as follows:

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
```

The `json:"name,omitempty"` syntax is a **struct tag** that controls JSON behavior. The first part (`"name"`) renames the field in JSON output. The `omitempty` option excludes the field if it has a zero value (empty string, nil slice, 0, etc.). We'll explore struct tags in detail later.

```go
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

An important note while working with either `Marshal` or `Unmarshal` functions is that the initial letter of struct fields must be uppercase, otherwise Go cannot export them for the JSON encoder to access.

The type of `byteSlice` variable is `[]byte` and by referring to the [Go language specification](https://go.dev/ref/spec#Types), we see that `byte` is an alias for `uint8` (the set of all unsigned 8-bit integers from 0 to 255). To convert a `[]byte` to human-readable format, we cast it to `string`: `string(byteSlice)` produces a string representation of the byte slice. Now let's create a file named `main_test.go` to add tests for our function:

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
			name:         "invalid cyclic data structure input",
			input:        node1,
			errorMessage: "json: unsupported value: encountered a cycle via *main.Node",
		},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			result, err := jsonEncoder(test.input)

			// Check error expectations
			if test.errorMessage != "" {
				if err == nil {
					t.Fatalf("expected error %q but got none", test.errorMessage)
				}
				if err.Error() != test.errorMessage {
					t.Errorf("expected error %q but received %q", test.errorMessage, err.Error())
				}
			} else if err != nil {
				t.Fatalf("unexpected error: %v", err)
			}

			// Check output only if no error expected
			if test.errorMessage == "" && result != test.output {
				t.Errorf("expected %q, but received %q", test.output, result)
			}
		})
	}
}
```

This table-driven test validates different scenarios: successful encoding, handling of zero values with `omitempty`, and the error case when encountering circular data structures.

## How to Convert JSON to Struct

In order to convert a JSON format to a struct, we need to use the `Unmarshal` function:

```go
func Unmarshal(data []byte, v any) error
```

This function receives input data of a slice of bytes (`[]byte`) as the first parameter and a pointer to a struct as the second one. If this process results in any error, `Unmarshal` returns it. In the following snippet, we have created a function called `jsonDecoder` which utilizes the `Unmarshal` function:

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
	lang, err := jsonDecoder(input)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println(lang)
}
```

It outputs:

```text
{Go [Cloud And Network Services Web Development Command-line Interfaces DevOps And Site Reliability]}
```

In the above program, we are converting the `input` param to `[]byte` type. If we print the type of `[]byte(input)` by calling `fmt.Printf("%T\n", []byte(input))`, we get `[]uint8`. As mentioned earlier, `byte` is an alias for `uint8`, so both `Marshal` and `Unmarshal` work with the same underlying type: `[]uint8`. Now let's add some tests to check the correctness of the `jsonDecoder` function:

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

			// Check error expectations
			if test.errorMessage != "" {
				if err == nil {
					t.Fatalf("expected error %q but got none", test.errorMessage)
				}
				if err.Error() != test.errorMessage {
					t.Fatalf("expected error %q but received %q", test.errorMessage, err.Error())
				}
			} else if err != nil {
				t.Fatalf("unexpected error: %v", err)
			}

			// Check output only if no error expected
			if test.errorMessage == "" && !reflect.DeepEqual(result, test.output) {
				t.Errorf("expected %v, but received %v", test.output, result)
			}
		})
	}
}
```

## When to Use `Encode` and `Decode` Functions

So far we have seen that if we have a struct and want to convert it to JSON, we can use the `Marshal` function, and when we want to convert a JSON object back to a struct, we can use the `Unmarshal` function. The `Encode` and `Decode` methods serve the same purposes respectively, but they work with streams of data rather than in-memory byte slices.

### Marshal/Unmarshal vs Encode/Decode

**Use Marshal/Unmarshal when:**
- You have the entire JSON payload already in memory (as a `[]byte` or string)
- Working with small to medium-sized JSON documents
- You need the simplest API for one-shot conversions

**Use Encode/Decode when:**
- Reading from or writing to I/O streams (files, network connections, HTTP requests/responses)
- Processing large JSON documents that shouldn't be loaded entirely into memory
- Working with streaming data or multiple JSON objects in sequence
- You want to avoid intermediate buffer allocations

**Performance considerations:** `Marshal` must allocate a complete `[]byte` buffer to hold the entire encoded result before returning. `Encode` writes incrementally to the underlying `io.Writer`, which can be more memory-efficient for large payloads. Similarly, `Unmarshal` requires the entire JSON document in memory as a `[]byte`, while `Decode` can read from streams. For large payloads or high-throughput scenarios, streaming with `Encode`/`Decode` reduces memory pressure and can improve performance by avoiding large buffer allocations.

### Understanding io.Writer and io.Reader Interfaces

To understand how `Encode` and `Decode` work, we need to know about the [`io.Writer`](https://pkg.go.dev/io#Writer) interface in Go.

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

As shown above, `Writer` is an interface, meaning any type with the `Write` method satisfies it and can be used wherever this interface is needed. For example, the [`json.NewEncoder`](https://pkg.go.dev/encoding/json#NewEncoder) function requires a type that implements `io.Writer` as its input parameter:

```go
func NewEncoder(w io.Writer) *Encoder
```

The [`os.File`](https://pkg.go.dev/os#File) type has a [`Write`](https://pkg.go.dev/os#File.Write) method, meaning it satisfies the `io.Writer` interface. Therefore, any variable of type `*os.File` can be passed to `NewEncoder`. Here's a practical example:

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
	if err := encoder.Encode(l); err != nil {
		fmt.Println("Error encoding JSON:", err)
		return
	}
}
```

Similarly, the [`json.NewDecoder`](https://pkg.go.dev/encoding/json#NewDecoder) function accepts any type that implements the [`io.Reader`](https://pkg.go.dev/io#Reader) interface with this signature:

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

Since [`os.Open`](https://pkg.go.dev/os#Open) returns a `*os.File` and this type has a [`Read`](https://pkg.go.dev/os#File.Read) method with the following signature:

```go
func (f *File) Read(b []byte) (n int, err error)
```

We can pass it directly to the `NewDecoder` function as shown below:

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

## Real-World Example: HTTP Handler

Here's a practical example showing when to use `Marshal` vs `Encode` in an HTTP handler:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
)

type Response struct {
	Status  string `json:"status"`
	Message string `json:"message"`
	Data    any    `json:"data,omitempty"`
}

// Bad: Uses Marshal with unnecessary allocation
func handlerWithMarshal(w http.ResponseWriter, r *http.Request) {
	resp := Response{
		Status:  "success",
		Message: "User created",
		Data:    map[string]string{"id": "123", "name": "Alice"},
	}

	// Marshal creates []byte in memory
	jsonBytes, err := json.Marshal(resp)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	w.Write(jsonBytes) // Then writes to response
}

// Good: Uses Encode to write directly to response
func handlerWithEncode(w http.ResponseWriter, r *http.Request) {
	resp := Response{
		Status:  "success",
		Message: "User created",
		Data:    map[string]string{"id": "123", "name": "Alice"},
	}

	w.Header().Set("Content-Type", "application/json")

	// Encode writes directly to http.ResponseWriter
	if err := json.NewEncoder(w).Encode(resp); err != nil {
		log.Printf("Error encoding response: %v", err)
		// Note: Headers already sent, can't return error to client
	}
}

func main() {
	http.HandleFunc("/marshal", handlerWithMarshal)
	http.HandleFunc("/encode", handlerWithEncode)

	fmt.Println("Server starting on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Key takeaway:** In HTTP handlers, prefer `Encode` to write directly to `http.ResponseWriter` rather than marshaling to `[]byte` first. This is more efficient and idiomatic. Use `Marshal` when you need the JSON as a byte slice for other purposes (logging, signing, storing).

## Best Practices and Common Pitfalls

### Struct Tags

The `json` struct tag controls JSON encoding behavior:

```go
type User struct {
    ID       int    `json:"id"`                    // Rename field in JSON
    Name     string `json:"name,omitempty"`        // Omit if zero value
    Password string `json:"-"`                     // Never include in JSON
    Age      int    `json:"age,string"`            // Encode number as string
}

type Admin struct {
    User     `json:",inline"`                      // Flatten embedded struct fields
    Role     string `json:"role"`
}
```

**Production tip:** Use `omitempty` for optional fields to produce cleaner JSON output, but be aware that it removes fields with zero values. If you need to distinguish between an absent field and a zero value (e.g., `false` vs absent for booleans), use pointer fields instead. Never expose sensitive fields like passwords—use `json:"-"` to exclude them.

The `json:",inline"` tag (also written as an empty name) flattens embedded struct fields into the parent. An `Admin` with embedded `User` produces `{"id": 1, "name": "Alice", "role": "admin"}` instead of `{"User": {"id": 1, "name": "Alice"}, "role": "admin"}`.

### Handling Unknown Fields

By default, `Unmarshal` ignores JSON fields that don't match your struct. To catch typos or enforce strict validation:

```go
decoder := json.NewDecoder(reader)
decoder.DisallowUnknownFields() // Returns error if JSON has extra fields
err := decoder.Decode(&myStruct)
```

### Pointer Fields for Distinguishing Null vs Absent

Use pointer fields when you need to distinguish between a field being absent, null, or having a zero value:

```go
type Config struct {
    Timeout *int `json:"timeout,omitempty"` // nil pointer omitted from JSON
                                              // non-nil pointer includes value: {"timeout": 0}
}
```

With `omitempty`, a `nil` pointer is excluded from JSON entirely, while a pointer to zero (or any value) is included in the output.

### Performance Considerations

1. **Use json.RawMessage for delayed parsing:** When you don't need to parse all fields immediately, use `json.RawMessage` to defer unmarshaling expensive nested structures.
2. **Streaming for large arrays:** When processing large JSON arrays, decode elements one at a time using `Decoder` rather than loading the entire array into memory with `Unmarshal`.
3. **Avoid unnecessary allocations:** Marshal creates a new `[]byte` on each call. If you're writing directly to an `io.Writer` (like an HTTP response), use `Encode` instead to avoid the intermediate buffer.

### Common Errors

- **Unexported fields are ignored:** Only fields starting with uppercase letters are encoded/decoded.
- **Circular references return errors:** `Marshal` cannot handle circular data structures and returns an error like "encountered a cycle via *Type" (as shown in our test case).
- **Type mismatches silently use zero values:** If JSON contains a string but your struct expects an int, `Unmarshal` sets the field to its zero value (0) and continues without error. Use `Decoder.DisallowUnknownFields()` for stricter validation.
- **Interface{} loses type information:** Unmarshaling into `interface{}` gives you generic types (`map[string]interface{}`, `[]interface{}`, `float64`, `string`, `bool`, `nil`) rather than your custom types. JSON numbers always become `float64`.
