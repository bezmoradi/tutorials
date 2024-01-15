# Go > Testing

-   To write a test, create a new file ending in `_test.go`. The other thing is that even test files need to belong to a package such as `package main`. First let's create a function called `concatTwoStrings()` in `main.go` file:

```go
package main

func main() {

}

func concatTwoStrings(s1 string, s2 string) string {
	return s1 + s2
}
```

-   Next, let's create a file called `main_test.go` as follows:

```go
package main

import "testing"

func TestConcatTwoStrings(t *testing.T) {
	result := concatTwoStrings("Bez", "Mor")

	if result != "BezMor" {
		t.Errorf("Expected BezMore but got %v", result)
	}
}
```

-   We can call the test function whatever we want but the VSCode extension asks us to start the name with `Test`
-   What `%v` does is that it attaches the variable which comes as the second argument to the error text string

## Test Coverage

-   In order to check the coverage metrics, you can run the `test` command with `-cover` flag as follows:

```go
$ go test -cover
```

## Integration Tests

-   Let's say we have the following folder structure:

```text
.
├── go.mod
├── main.go
└── src
    └── api
        ├── math
        │   ├── math-operations.go
        │   └── math-operations_test.go
        └── services
            ├── math-service.go
            └── math-service_test.go
```

-   The `math-operations.go` file includes:

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

-   and the `math-service.go` file includes:

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

-   Basically, `math-service.go` depends on `math-operations.go` and if we are about to test `math-service.go`, we have to deal with an Integration test as follows:

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

-   Writing integration tests is no different from unit tests; the only difference is that inside the main function, we are delegating part of the algorithm to another function and as long as we are not mocking that other function, we are writing an integration test

## Benchmarks

-   I we want to test the efficiency of the same functionality but with different algorithms, we can use the benchmarking functionality that is built-in in GO

```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		add(1, 2)
	}
}
```

-   As shown above, the test name starts with `Benchmark` and the type of the input param is `*testing.B` which stands for "Benchmark".
-   Benchmarks are not run by default; so in order to run them we have to use the `go test -bench=.` or `go test -bench .` commands:

```text
$ go test -bench .
goos: darwin
goarch: arm64
pkg: playground/src/api/services
BenchmarkAdd-8   	1000000000	         0.5125 ns/op
PASS
ok  	playground/src/api/services	0.756s
```

-   As shown above, our function is called `1000000000` times and on average it takes `0.5125` nano seconds per operation

## Test Coverage

-   To see what percentage of our codebase is covered using tests, we can use the `-cover` flag as follows:

```text
& go test -cover
PASS
coverage: 50.0% of statements
ok  	playground	0.169s
```

-   As shown above, `50%` of our main code is covered using tests which is not ideal!
-   Also we can output the result into a file then open that file inside the browser:

```text
& go test -coverprofile output.txt
```

-   The above command created a file called `output.txt` including the coverage results. In order to convert that file into an HTML document and show it inside the browser, we need to run the following command:

```text
& go tool cover -html=output.txt
```

-   As soon as you hit enter, a new browser tab will be popped up showing the coverage results
