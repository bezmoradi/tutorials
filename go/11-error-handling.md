# Go > Error Handling

Go designers believe that coupling exceptions to a control structure, as in the try-catch-finally idiom, results in convoluted code. It also tends to encourage programmers to label too many ordinary errors, such as failing to open a file, as exceptional which in reality it isn't. In Go, the following `error` interface defines what can be considered as an `error` type:

```go
type error interface {
	Error() string
}
```

In other words, any type that has an `Error` method attached to it is considered as an `error` in Go.

## An Intro to The `errors` Package

In Go, the `https://pkg.go.dev/errors` package can be used for creating custom errors using the `New` function:

```go
func New(text string) error
```

As shown above, this function receives a string as an input and returns an `error`. If we take a peek at the source of the `New` function, we have:

```go
func New(text string) error {
	return &errorString{text}
}

type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

What the `New` function does is that it creates a variable of type `errorString` and passes its input param as its `s` field. For the `errorString` type to be of type `error` interface, it needs to have an `Error` method which returns a string.

## Difference between `Exit` and `Panic`

The `log.Fatal` function calls `os.Exit(1)` and the program terminates immediately while deferred functions are not run:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func neverRuns() {
	fmt.Println("This won't run!")
}

func main() {
	defer neverRuns()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		log.Fatal(err)
	}
}
```

In fact, `log.Fatal` is equivalent to `fmt.Print` followed by `os.Exit(1)`:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	defer neverRuns()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		fmt.Print(err)
		os.Exit(1)
	}
}
```

Whereas the `log.Panic` function implies that there is an issue but still we have a chance to recover our program from terminating:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func deferredFunction() {
	fmt.Println("this is a deferred function called")
}

func main() {
	defer deferredFunction()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		log.Panic(err)
	}
}
```

The key point here is that contrary to `log.Fatal`, any deferred function calls will be run then the program exits. The equivalent using the `panic` function is as follows:

```go
func main() {
	defer deferredFunction()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		panic(err)
	}
}
```

The `recover` function is built into Go that regains the control of a panicking goroutine which is only useful inside deferred functions. In that case, calling `recover` will capture the value given to the `panic` function and resume the normal execution. In Go, `panic` and `recover` are used to manage unexpected situations in your code, especially during runtime errors:

```go
package main

import (
	"fmt"
	"os"
)

func deferredFunction() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recovered from panic", r)
		}
	}()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		panic(err)
	}
}

func main() {
	defer deferredFunction()
}
```

The `recover` function returns the value that was passed to the `panic`. If there is no `panic` or if `recover` is called outside the deferred function or after the panic has propagated outside that function, it returns `nil`.

## Creating Custom Errors

We can create custom errors as long as they adhere to the `error` interface. Let's create a factory function for creating errors:

```go
package main

import (
	"errors"
	"fmt"
)

func errorGenerator(e error) string {
	packageName := "main"
	return packageName + " -> " + e.Error()
}

func main() {
	fmt.Println(
		errorGenerator(
			errors.New("this is an error"),
		),
	) // main -> this is an error
}
```

The `errorGenerator` function receives an input param of type `error` meaning any variable that adheres to the `error` interface can be used. Let's see that in action:

```go
package main

import (
	"fmt"
)

type customError struct {
	s        string
	fileName string
}

func (c customError) Error() string {
	return fmt.Sprintf("%v -> %v", c.fileName, c.s)
}

func errorGenerator(e error) string {
	packageName := "main"
	return packageName + " -> " + e.Error()
}

func main() {
	err := customError{
		fileName: "main.go",
		s:        "this is an error",
	}

	fmt.Println(errorGenerator(err)) // main -> main.go -> this is an error
}
```

Technically, as the `customError` struct has a function called `Error`, it can be considered as an `error` type; so it can be easily used as an input param for the `errorGenerator` function.
