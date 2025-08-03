# Go > Testing

First let's create a function called `concatTwoStrings` in `main.go` file:

```go
package main

import "fmt"

func concatTwoStrings(s1 string, s2 string) string {
	return s1 + s2
}

func main() {
	fmt.Println(concatTwoStrings("Go", "lang"))
}
```

Next, let's create a file called `main_test.go` as follows:

```go
package main

import "testing"

func TestConcatTwoStrings(t *testing.T) {
	result := concatTwoStrings("Go", "lang")
	if result != "Golang" {
		t.Errorf("should return Golang but got %v", result)
	}
}
```

We can call the test function whatever we want but the VSCode extension asks us to start the name with `Test`. By convention, we can add the name of the function being tested like `TestConcatTwoStrings` (Another valid name for our test function would be `Test_concat_two_strings`)

## Defining Test Cases

Within the `TestConcatTwoStrings` function, we can define multiple test cases using `t.Run`. Each test case has a description (a string) and a function that contains the actual test logic:

```go
func TestConcatTwoStrings(t *testing.T) {
	t.Run(`concat "Go" and "lang"`, func(t *testing.T) {
		result := concatTwoStrings("Go", "lang")
		if result != "Golang" {
			t.Errorf("should return Golang but got %v", result)
		}
	})

	t.Run(`concat "Hello" and "World"`, func(t *testing.T) {
		result := concatTwoStrings("Hello", "World")
		if result != "HelloWorld" {
			t.Errorf("should return HelloWorld but got %v", result)
		}
	})
}
```

## Integration Tests

Let's say we have the following folder structure:

```text
.
├── go.mod
├── main.go
└── internal
    └── api
        ├── math
        │   ├── math-operations.go
        │   └── math-operations_test.go
        └── services
            ├── math-service.go
            └── math-service_test.go
```

The `math-operations.go` file includes:

```go
package math

func MathOperations(s1, s2 int, operation string) int {
	switch operation {
	case "ADD":
		return s1 + s2
	case "SUBTRACT":
		return s1 - s2
	default:
		return 0
	}
}
```

And the `math-service.go` file includes:

```go
package services

import "playground/src/api/math"

func add(l, r int) int {
	return math.MathOperations(l, r, "ADD")
}

func subtract(l, r int) int {
	return math.MathOperations(l, r, "SUBTRACT")
}
```

Basically, `math-service.go` depends on `math-operations.go` and if we are about to test `math-service.go`, we have to deal with an Integration Test as follows:

```go
package services

import "testing"

func TestAdd(t *testing.T) {
	result := add(1, 2)
	if result != 3 {
		t.Errorf("The result must be 3 but got %v", result)
	}
}

func TestSubtract(t *testing.T) {
	result := add(3, 1)
	if result != 2 {
		t.Errorf("The result must be 2 but got %v", result)
	}
}
```

Writing integration tests is no different from unit tests; the only difference is that we are delegating part of the algorithm to another function and as long as we are not mocking that other function, we are writing an integration test.

## Benchmarks

If we want to test the efficiency of the same functionality but with different algorithms, we can use the benchmarking functionality that is built-in in Go:

```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		add(1, 2)
	}
}
```

As shown above, the test name starts with `Benchmark` and the type of the input param is `*testing.B` which stands for "Benchmark". Benchmarks are not run by default; so in order to run them we have to use the `go test -bench=.` or `go test -bench .` commands:

```text
$ go test -bench .
goos: darwin
goarch: arm64
pkg: playground/src/api/services
BenchmarkAdd-8   	1000000000	         0.5125 ns/op
PASS
ok  	playground/src/api/services	0.756s
```

Our function is called `1,000,000,000` times and on average it takes `0.5125` nano seconds per operation

## Test Coverage

To see what percentage of our codebase is covered using tests, we can use the `-cover` flag as follows:

```sh
& go test -cover
PASS
coverage: 50.0% of statements
ok  	playground	0.169s
```

As shown above, `50%` of our code is covered using tests. Also we can output the result into a file then open that file inside the browser. To do that, a new file called `output.txt` will be created by running the following command:

```sh
& go test -coverprofile output.txt
```

The `output.txt` file includes the coverage results. In order to load that file into as an HTML document and show it inside the browser, we need to run the following command:

```sh
& go tool cover -html=output.txt
```

As soon as you hit enter, a new browser tab will be popped up showing the coverage results.

## Table Test

A table test is a testing technique used to systematically test a function or method with multiple inputs and expected outputs. Instead of writing individual test cases for each input/output pair, you organize the test data in a tabular format (often a slice of structs or arrays) and iterate over that to run the tests. As an exercise, we are going to created a function called `stringToSlug` which is responsible for replacing any whitespace with dash and remove any non-roman and non-digit characters:

```go
package main

import (
	"errors"
	"fmt"
	"regexp"
	"strings"
)

func stringToSlug(s string) (string, error) {
	s = regexp.
		MustCompile("[^a-z0-9]+").
		ReplaceAllString(s, "-")

	s = strings.TrimSuffix(s, "-")

	if s == "" {
		return "", errors.New("empty string not allowed")
	}

	return s, nil
}

func main() {
	slug, err := stringToSlug("hello world 123 !@# こんにちは")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	fmt.Println(slug) // hello-world-123
}
```

Now it's time to write a table test for it:

```go
package main

import "testing"

func TestStringToSlug(t *testing.T) {
	testCases := []struct {
		testName     string
		input        string
		output       string
		errorMessage string
	}{
		{
			testName:     "valid input",
			input:        "hello world",
			output:       "hello-world",
			errorMessage: "",
		},
		{
			testName:     "complex input",
			input:        "hello world 123 !@#",
			output:       "hello-world-123",
			errorMessage: "",
		},
		{
			testName:     "non-english & english input",
			input:        "hello world こんにちは",
			output:       "hello-world",
			errorMessage: "",
		},
		{
			testName:     "empty string",
			input:        "",
			output:       "",
			errorMessage: "empty string not allowed",
		},
		{
			testName:     "non-english string",
			input:        "こんにちは",
			output:       "",
			errorMessage: "empty string not allowed",
		},
	}

	for _, test := range testCases {
		slug, err := stringToSlug(test.input)
		if err != nil && err.Error() != test.errorMessage {
			t.Errorf("expected %v but got %v", test.errorMessage, err.Error())
		}
		if test.errorMessage == "" && test.output != slug {
			t.Errorf("expected %v but got %v", test.output, slug)
		}
	}
}
```

## How to Run Tests

In order to run all tests in the current directory, we can do:

```sh
$ go test
```

To test a single test we can:

```sh
$ go test -run TestMyFunction
```

To test groups of tests, first we need to split tests into groups by some naming convention. For example, by adding the `TestGroup` as a prefix to test names like `TestGroupFuncA` and `TestGroupFuncB` then while running the test enter:

```sh
$ go test -run TestGroup
```

Behind the scenes, it works like a regex with the `TestGroup*` rule which runs all tests with such a prefix. And to run all tests in all packages we have:

```sh
$ go test ./...
```

## How to Create A Setup File?

In order to stick to the DRY (Don't Repeat Yourself) principle, in Go we can create a special file called `setup_test.go` as follows which will be responsible for preparing some stuff that can be used throughout all other test files:

```go
package main

import (
	"os"
	"testing"
)

const GlobalVariable string = "John"

func TestMain(m *testing.M) {
	os.Exit(m.Run())
}
```

From now on, as soon as we run `go test ./...`, the Go test runner first runs this file then all other tests; that's why the `GlobalVariable` constant will be available in all other tests.

## How to Separate Integration Tests from Unit Tests?

Simply by adding `//go:build integration` at the very first line of your test files, the Go test runner would consider that file as an integration test and by default won't run it. In order to run it, we should add the following flag:

```sh
$ go test -tags=integration ./...
```

## How to Add Helper Functions to Test Files?

We can simply move any code duplication into a helper function. To see how we can do that, let's consider the previously-reviewed `main.go` file:

```go
package main

import "fmt"

func concatTwoStrings(s1 string, s2 string) string {
	return s1 + s2
}

func main() {
	fmt.Println(concatTwoStrings("Go", "lang"))
}
```

And the `main_test.go` file is as follows:

```go
package main

import "testing"

func TestConcatTwoStrings(t *testing.T) {
	result := concatTwoStrings("Go", "lang")
	assert(t, result, "Golang")
}

func assert(t *testing.T, got, want string) {
	t.Helper()

	if got != want {
		t.Errorf("expected %v but got %v", want, got)
	}
}
```

The important thing about the `assert` helper function is calling `t.Helper()` which tells the test suite that this method is a helper. By doing this when it fails the line number reported will be in our function call rather than inside our test helper. This will help other developers track down problems easier.

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

The `Pipe` function creates a communication mechanism that allows one process to send data to another process. In the above example, it's used for capturing the output of a function, as you're doing in your test. It returns two `*os.File` values, one for the read end (`r`) and one for the write end (`w`) of the pipe.

## Links

-   https://quii.gitbook.io/learn-go-with-tests
