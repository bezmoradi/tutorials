# Go > Error Handling

Go designers believe that coupling exceptions to a control structure, as in the try-catch-finally idiom, results in convoluted code. It also tends to encourage programmers to label too many ordinary errors, such as failing to open a file, as exceptional, which in reality they aren't. In Go, the following `error` interface defines what can be considered as an `error` type:

```go
type error interface {
	Error() string
}
```

In other words, any type that has an `Error` method attached to it is considered as an `error` in Go.

## An Intro to the `errors` Package

In Go, the `errors` package can be used for creating custom errors using the `New` function:

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

The `New` function creates a variable of type `errorString` and passes its input param as its `s` field. For the `errorString` type to be of type `error` interface, it needs to have an `Error` method which returns a string.

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

Whereas the `log.Panic` function implies that there is an issue, but we still have a chance to recover our program from terminating:

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

The key point here is that unlike `log.Fatal`, any deferred function calls will be run before the program exits. The equivalent using the `panic` function is as follows:

```go
func main() {
	defer deferredFunction()

	_, err := os.Open("NON_EXISTING_FILE")
	if err != nil {
		panic(err)
	}
}
```

The `recover` function is built into Go that regains control of a panicking goroutine, which is only useful inside deferred functions. In that case, calling `recover` will capture the value given to the `panic` function and resume the normal execution. In Go, `panic` and `recover` are used to manage unexpected situations in your code, especially during runtime errors:

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

Technically, as the `customError` struct has a method called `Error`, it can be considered as an `error` type; so it can be easily used as an input param for the `errorGenerator` function.

## Error Wrapping

Starting with Go 1.13, the language introduced error wrapping, which allows you to add context to errors while preserving the original error for inspection. This is a crucial feature for building maintainable applications where errors need to be traced through multiple layers.

### Using `fmt.Errorf` with `%w`

The `%w` verb in `fmt.Errorf` wraps an error, making the original error available for inspection using `errors.Is()` and `errors.As()`:

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func readConfig(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		// Wrap the error with context
		return fmt.Errorf("failed to read config: %w", err)
	}
	return nil
}

func initializeApp() error {
	err := readConfig("config.json")
	if err != nil {
		// Wrap again with more context
		return fmt.Errorf("app initialization failed: %w", err)
	}
	return nil
}

func main() {
	err := initializeApp()
	if err != nil {
		fmt.Println(err)
		// Output: app initialization failed: failed to read config: open config.json: no such file or directory
	}
}
```

When you wrap an error, the resulting error contains the full chain of context. Each layer adds information about where and why the error occurred.

### Unwrapping Errors

The `errors` package provides an `Unwrap` function to access the underlying error:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	originalErr := errors.New("original error")
	wrappedErr := fmt.Errorf("wrapped: %w", originalErr)

	// Unwrap to get the original error
	unwrapped := errors.Unwrap(wrappedErr)
	fmt.Println(unwrapped) // Output: original error

	// Unwrapping a non-wrapped error returns nil
	notWrapped := errors.New("not wrapped")
	result := errors.Unwrap(notWrapped)
	fmt.Println(result == nil) // Output: true
}
```

### When to Wrap vs Return Directly

Use `%w` when you want to preserve the original error for comparison or inspection. Use `%v` when you want to include the error message but don't need to expose the error type:

```go
package main

import (
	"fmt"
	"os"
)

func processFile(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		// Use %w to preserve the error for inspection
		return fmt.Errorf("cannot process file %s: %w", filename, err)
	}
	return nil
}

func logError(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		// Use %v to hide the internal error type
		return fmt.Errorf("logging failed for %s: %v", filename, err)
	}
	return nil
}

func main() {
	// The wrapped error can be inspected
	err1 := processFile("data.txt")
	if err1 != nil {
		fmt.Println("Wrapped:", err1)
	}

	// The error message is included but not wrapped
	err2 := logError("log.txt")
	if err2 != nil {
		fmt.Println("Not wrapped:", err2)
	}
}
```

Wrapping errors makes them part of your API. If external code depends on checking specific error types, wrapping allows that. If you want to hide implementation details, avoid wrapping.

## Inspecting Errors with `errors.Is()` and `errors.As()`

When working with wrapped errors, you need special functions to inspect them. The `errors.Is()` and `errors.As()` functions traverse the error chain to find matches.

### Using `errors.Is()`

The `errors.Is()` function checks if an error matches a specific error value anywhere in the error chain. This is the modern replacement for direct error comparison with `==`:

```go
package main

import (
	"errors"
	"fmt"
	"io/fs"
	"os"
)

func readFile(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("failed to open file: %w", err)
	}
	return nil
}

func main() {
	err := readFile("nonexistent.txt")

	// Check if the error is or wraps fs.ErrNotExist
	if errors.Is(err, fs.ErrNotExist) {
		fmt.Println("File does not exist")
	}

	// Direct comparison with == won't work with wrapped errors
	if err == fs.ErrNotExist {
		fmt.Println("This won't print")
	}
}
```

The `errors.Is()` function performs a depth-first traversal of the error chain, checking each error against the target. This means it works even when errors are wrapped multiple times.

### Using `errors.As()`

The `errors.As()` function finds the first error in the chain that matches a specific type and assigns it to the target variable. This is useful when you need to access fields or methods of a specific error type:

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func processFile(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("processing failed: %w", err)
	}
	return nil
}

func main() {
	err := processFile("config.json")

	// Check if the error is or contains a *os.PathError
	var pathErr *os.PathError
	if errors.As(err, &pathErr) {
		fmt.Printf("Failed operation: %s\n", pathErr.Op)
		fmt.Printf("Path: %s\n", pathErr.Path)
		fmt.Printf("Underlying error: %v\n", pathErr.Err)
	}
}
```

Output:
```text
Failed operation: open
Path: config.json
Underlying error: no such file or directory
```

### Practical Example: HTTP Error Handling

Here's a practical example showing how `errors.Is()` and `errors.As()` work together in a real application:

```go
package main

import (
	"errors"
	"fmt"
	"net/http"
)

// Custom error types
var (
	ErrNotFound      = errors.New("resource not found")
	ErrUnauthorized  = errors.New("unauthorized access")
	ErrInternalError = errors.New("internal server error")
)

type ValidationError struct {
	Field string
	Issue string
}

func (v ValidationError) Error() string {
	return fmt.Sprintf("validation error: %s - %s", v.Field, v.Issue)
}

func getUser(id int) error {
	if id < 0 {
		return ValidationError{Field: "id", Issue: "must be positive"}
	}
	if id == 0 {
		return fmt.Errorf("user lookup failed: %w", ErrNotFound)
	}
	return nil
}

func handleRequest(userID int) {
	err := getUser(userID)
	if err == nil {
		fmt.Println("User found successfully")
		return
	}

	// Check for specific sentinel errors
	if errors.Is(err, ErrNotFound) {
		fmt.Println("HTTP 404: User not found")
		return
	}

	if errors.Is(err, ErrUnauthorized) {
		fmt.Println("HTTP 401: Unauthorized")
		return
	}

	// Check for specific error types
	var validationErr ValidationError
	if errors.As(err, &validationErr) {
		fmt.Printf("HTTP 400: Invalid %s - %s\n", validationErr.Field, validationErr.Issue)
		return
	}

	// Default case
	fmt.Println("HTTP 500: Internal server error")
}

func main() {
	handleRequest(-1) // HTTP 400: Invalid id - must be positive
	handleRequest(0)  // HTTP 404: User not found
	handleRequest(42) // User found successfully
}
```

### Key Differences Between `Is` and `As`

- **`errors.Is(err, target)`**: Checks if any error in the chain equals the target error value (sentinel error)
- **`errors.As(err, &target)`**: Checks if any error in the chain is of the target type and assigns it

Use `Is` when you care about a specific error value. Use `As` when you need to access fields or methods of a specific error type.

## Sentinel Errors

Sentinel errors are predefined error values declared at the package level. They provide a shared signal across your codebase for specific error conditions. The Go standard library uses sentinel errors extensively, such as `io.EOF`, `sql.ErrNoRows`, and `fs.ErrNotExist`.

### Defining Sentinel Errors

Sentinel errors are typically declared as package-level variables:

```go
package database

import "errors"

var (
	ErrNotFound    = errors.New("record not found")
	ErrDuplicate   = errors.New("duplicate record")
	ErrInvalidData = errors.New("invalid data")
)

func GetUser(id int) error {
	// Simulate database lookup
	if id == 0 {
		return ErrNotFound
	}
	if id < 0 {
		return ErrInvalidData
	}
	return nil
}
```

### Using Sentinel Errors

Callers can check for sentinel errors using `errors.Is()`:

```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

func findItem(id int) error {
	if id == 0 {
		return ErrNotFound
	}
	return nil
}

func main() {
	err := findItem(0)

	if errors.Is(err, ErrNotFound) {
		fmt.Println("Item doesn't exist, creating new one...")
		// Handle the specific case
	}
}
```

### Wrapping Sentinel Errors

You can wrap sentinel errors to add context while preserving the ability to check for them:

```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

func getUserByID(id int) error {
	if id == 0 {
		return fmt.Errorf("user %d: %w", id, ErrNotFound)
	}
	return nil
}

func main() {
	err := getUserByID(0)

	// Still works with wrapped errors
	if errors.Is(err, ErrNotFound) {
		fmt.Println("User not found")
	}

	fmt.Println(err) // Output: user 0: not found
}
```

### When to Use Sentinel Errors

Sentinel errors are best suited for:

1. **Well-defined, stable error conditions**: Conditions that won't change over time
2. **Shared error states**: When multiple packages need to recognize the same error
3. **Public APIs**: When external code needs to handle specific errors differently

Examples from the standard library:
```go
package main

import (
	"errors"
	"fmt"
	"io"
	"strings"
)

func main() {
	reader := strings.NewReader("Hello")
	buf := make([]byte, 10)

	// First read gets the data
	n, err := reader.Read(buf)
	fmt.Printf("Read %d bytes: %s\n", n, buf[:n])

	// Second read hits EOF
	_, err = reader.Read(buf)
	if errors.Is(err, io.EOF) {
		fmt.Println("Reached end of stream")
	}
}
```

### Performance Considerations

While sentinel errors are convenient, checking them with `errors.Is()` has performance implications. The function traverses the entire error chain, which can be slower than direct comparison. For performance-critical code paths with high error rates, consider:

1. **Using boolean returns** alongside errors for hot paths
2. **Avoiding deeply nested error wrapping** in tight loops
3. **Profiling** to identify if error checking is actually a bottleneck

In most applications, the performance impact is negligible, and the clarity of sentinel errors outweighs the cost.

### Best Practices for Sentinel Errors

1. **Naming convention**: Prefix with `Err` (e.g., `ErrNotFound`, `ErrTimeout`)
2. **Package-level scope**: Declare at package level, not inside functions
3. **Documentation**: Document what each sentinel error represents
4. **Stability**: Don't change sentinel error definitions once published
5. **Use sparingly**: Only create sentinel errors for conditions that callers genuinely need to distinguish

```go
package user

import "errors"

// ErrNotFound indicates the requested user was not found in the database.
// Callers can check for this error to distinguish between "not found" and
// other database errors.
var ErrNotFound = errors.New("user not found")

// ErrInvalidEmail indicates the provided email address is malformed.
var ErrInvalidEmail = errors.New("invalid email address")
```

## Idiomatic Error Handling Patterns

Go has established conventions for error handling that make code predictable and maintainable. Following these patterns makes your code easier to understand and work with.

### The Multiple Return Value Pattern

The most common Go pattern is returning an error as the last return value:

```go
package main

import (
	"errors"
	"fmt"
)

func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divide(10, 2)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Result:", result)
}
```

### Early Return Pattern

Check errors immediately and return early. This keeps the happy path left-aligned and makes the code flow easier to follow:

```go
package main

import (
	"fmt"
	"os"
)

func processFile(filename string) error {
	// Bad: nested error handling
	// file, err := os.Open(filename)
	// if err == nil {
	//     defer file.Close()
	//     data, err := io.ReadAll(file)
	//     if err == nil {
	//         // process data
	//     } else {
	//         return err
	//     }
	// } else {
	//     return err
	// }

	// Good: early returns
	file, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("failed to open file: %w", err)
	}
	defer file.Close()

	// Happy path continues here
	// More operations...
	return nil
}

func main() {
	if err := processFile("data.txt"); err != nil {
		fmt.Println("Error:", err)
	}
}
```

### Error Propagation

When a function can't handle an error, it should propagate it to the caller, typically adding context:

```go
package main

import (
	"errors"
	"fmt"
)

var ErrInvalidInput = errors.New("invalid input")

func validateInput(input string) error {
	if input == "" {
		return ErrInvalidInput
	}
	return nil
}

func processInput(input string) error {
	if err := validateInput(input); err != nil {
		return fmt.Errorf("input processing failed: %w", err)
	}
	// Process the input
	return nil
}

func handleRequest(input string) error {
	if err := processInput(input); err != nil {
		return fmt.Errorf("request handling failed: %w", err)
	}
	return nil
}

func main() {
	err := handleRequest("")
	if err != nil {
		fmt.Println(err)
		// Output: request handling failed: input processing failed: invalid input
	}
}
```

### Handling Errors in Loops

When processing multiple items, decide whether to stop at the first error or collect all errors:

```go
package main

import (
	"fmt"
)

// Stop at first error
func processItemsFailFast(items []string) error {
	for i, item := range items {
		if item == "" {
			return fmt.Errorf("invalid item at index %d", i)
		}
		// Process item
		fmt.Println("Processing:", item)
	}
	return nil
}

// Collect all errors
func processItemsCollectErrors(items []string) []error {
	var errs []error
	for i, item := range items {
		if item == "" {
			errs = append(errs, fmt.Errorf("invalid item at index %d", i))
			continue
		}
		// Process item
		fmt.Println("Processing:", item)
	}
	return errs
}

func main() {
	items := []string{"apple", "", "banana", ""}

	fmt.Println("Fail fast:")
	if err := processItemsFailFast(items); err != nil {
		fmt.Println("Error:", err)
	}

	fmt.Println("\nCollect errors:")
	errs := processItemsCollectErrors(items)
	if len(errs) > 0 {
		fmt.Println("Errors encountered:")
		for _, err := range errs {
			fmt.Println("-", err)
		}
	}
}
```

### Error Context Best Practices

Add context to errors to make debugging easier:

```go
package main

import (
	"fmt"
	"os"
)

func loadConfig(filename string) error {
	_, err := os.Open(filename)
	if err != nil {
		// Good: includes what failed and relevant context
		return fmt.Errorf("failed to load config from %s: %w", filename, err)
	}
	return nil
}

func initializeDatabase(connString string) error {
	// Simulate connection
	if connString == "" {
		// Good: specific error message
		return fmt.Errorf("database initialization: connection string is empty")
	}
	return nil
}

func startApplication() error {
	if err := loadConfig("config.json"); err != nil {
		return fmt.Errorf("application startup failed: %w", err)
	}

	if err := initializeDatabase("postgres://..."); err != nil {
		return fmt.Errorf("application startup failed: %w", err)
	}

	return nil
}

func main() {
	if err := startApplication(); err != nil {
		fmt.Println("Error:", err)
	}
}
```

### When to Use Panic

Use `panic` only for truly exceptional circumstances where the program cannot continue:

```go
package main

import "fmt"

func init() {
	// Panic in init() for invalid configuration
	// This prevents the application from running with bad config
	if criticalConfigMissing := true; criticalConfigMissing {
		panic("critical configuration is missing")
	}
}

func processData(data []int) {
	// Don't panic for expected error conditions
	// Bad: panic(errors.New("empty data"))

	// Good: return error
	if len(data) == 0 {
		fmt.Println("Warning: empty data")
		return
	}

	// Process data
}

func main() {
	// Panic is acceptable for programmer errors
	// that should never happen
	arr := []int{1, 2, 3}
	_ = arr[0] // Safe
	// _ = arr[10] // Would panic - index out of range

	processData([]int{})
}
```

### Error Handling in Goroutines

Errors in goroutines need special handling since they can't return to the caller:

```go
package main

import (
	"fmt"
	"sync"
)

func processWithChannel(items []string) <-chan error {
	errChan := make(chan error, len(items))
	var wg sync.WaitGroup

	for _, item := range items {
		wg.Add(1)
		go func(item string) {
			defer wg.Done()
			if item == "" {
				errChan <- fmt.Errorf("invalid item: %s", item)
				return
			}
			// Process item
			fmt.Println("Processed:", item)
		}(item)
	}

	go func() {
		wg.Wait()
		close(errChan)
	}()

	return errChan
}

func main() {
	items := []string{"apple", "", "banana"}

	errChan := processWithChannel(items)

	// Collect errors
	var errors []error
	for err := range errChan {
		errors = append(errors, err)
	}

	if len(errors) > 0 {
		fmt.Println("Errors occurred:")
		for _, err := range errors {
			fmt.Println("-", err)
		}
	}
}
```

## Error Handling Best Practices

Following these best practices will help you write robust, maintainable Go code with clear error handling.

### 1. Error Messages Should Be Actionable

Write error messages that help users or developers understand what went wrong and how to fix it:

```go
// Bad: vague error message
return errors.New("invalid input")

// Good: specific and actionable
return fmt.Errorf("invalid email address %q: missing @ symbol", email)
```

### 2. Error Messages Should Be Lowercase

Error messages should start with lowercase letters and not end with punctuation (unless it's a multi-sentence message). This follows Go conventions and allows errors to be wrapped with additional context:

```go
// Bad
errors.New("File not found.")
errors.New("Invalid input")

// Good
errors.New("file not found")
errors.New("invalid input")
```

### 3. Don't Log and Return Errors

Either handle an error (log it) or return it to the caller, but don't do both. This prevents duplicate log entries as the error bubbles up:

```go
// Bad: logs and returns
func processFile(filename string) error {
	f, err := os.Open(filename)
	if err != nil {
		log.Println("Error opening file:", err)
		return err // Caller might also log this
	}
	defer f.Close()
	return nil
}

// Good: just return
func processFile(filename string) error {
	f, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("failed to process %s: %w", filename, err)
	}
	defer f.Close()
	return nil
}

// Good: handle at the top level
func main() {
	if err := processFile("data.txt"); err != nil {
		log.Println("Error:", err)
	}
}
```

### 4. Use Custom Error Types for Rich Error Information

When you need to provide structured error information, define custom error types:

```go
package main

import "fmt"

type ValidationError struct {
	Field   string
	Value   interface{}
	Message string
}

func (e ValidationError) Error() string {
	return fmt.Sprintf("validation failed for %s (value: %v): %s",
		e.Field, e.Value, e.Message)
}

func validateAge(age int) error {
	if age < 0 {
		return ValidationError{
			Field:   "age",
			Value:   age,
			Message: "must be non-negative",
		}
	}
	if age > 150 {
		return ValidationError{
			Field:   "age",
			Value:   age,
			Message: "must be less than 150",
		}
	}
	return nil
}

func main() {
	if err := validateAge(-5); err != nil {
		fmt.Println(err)
		// Output: validation failed for age (value: -5): must be non-negative
	}
}
```

### 5. Consider Error Context at Each Layer

Each layer of your application should add relevant context:

```go
package main

import (
	"fmt"
	"os"
)

// Storage layer: basic error
func readFromDisk(path string) ([]byte, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, fmt.Errorf("disk read failed: %w", err)
	}
	return data, nil
}

// Business logic layer: add domain context
func loadUserProfile(userID string) ([]byte, error) {
	path := fmt.Sprintf("/data/users/%s.json", userID)
	data, err := readFromDisk(path)
	if err != nil {
		return nil, fmt.Errorf("failed to load profile for user %s: %w", userID, err)
	}
	return data, nil
}

// API layer: add request context
func handleGetProfile(userID string, requestID string) error {
	_, err := loadUserProfile(userID)
	if err != nil {
		return fmt.Errorf("request %s failed: %w", requestID, err)
	}
	return nil
}

func main() {
	err := handleGetProfile("user123", "req-456")
	if err != nil {
		fmt.Println(err)
		// Output: request req-456 failed: failed to load profile for user user123:
		//         disk read failed: open /data/users/user123.json: no such file or directory
	}
}
```

### 6. Don't Ignore Errors

Always handle errors explicitly, even if it's just to explicitly ignore them:

```go
// Bad: silently ignoring error
file.Close()

// Good: explicitly ignoring
_ = file.Close()

// Best: handle the error
if err := file.Close(); err != nil {
	log.Printf("failed to close file: %v", err)
}
```

### 7. Design Error Hierarchies Carefully

When designing packages, think carefully about which errors should be public:

```go
package database

import (
	"errors"
	"fmt"
)

// Public errors that callers should handle
var (
	ErrNotFound   = errors.New("record not found")
	ErrDuplicate  = errors.New("duplicate record")
	ErrConnection = errors.New("connection failed")
)

// Private error for internal use
var errInvalidQuery = errors.New("invalid query")

func Query(id string) error {
	// Internal errors are wrapped and not exposed
	if id == "" {
		return fmt.Errorf("%w: empty id", errInvalidQuery)
	}

	// Public errors can be returned directly or wrapped
	return ErrNotFound
}
```

### 8. Test Error Conditions

Always test that your code handles errors correctly:

```go
package main

import (
	"errors"
	"testing"
)

var ErrDivideByZero = errors.New("division by zero")

func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, ErrDivideByZero
	}
	return a / b, nil
}

func TestDivide(t *testing.T) {
	tests := []struct {
		name    string
		a, b    int
		want    int
		wantErr error
	}{
		{"normal division", 10, 2, 5, nil},
		{"divide by zero", 10, 0, 0, ErrDivideByZero},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := divide(tt.a, tt.b)
			if !errors.Is(err, tt.wantErr) {
				t.Errorf("divide() error = %v, wantErr %v", err, tt.wantErr)
				return
			}
			if got != tt.want {
				t.Errorf("divide() = %v, want %v", got, tt.want)
			}
		})
	}
}
```

### 9. Consider Using Third-Party Error Packages for Complex Needs

For applications requiring advanced error handling features (like stack traces), consider packages like:

- `github.com/pkg/errors`: Adds stack traces to errors
- `github.com/hashicorp/go-multierror`: Accumulate multiple errors
- `golang.org/x/sync/errgroup`: Handle errors from groups of goroutines

Example with errgroup:

```go
package main

import (
	"context"
	"fmt"
	"golang.org/x/sync/errgroup"
	"time"
)

func fetchUser(id int) error {
	if id < 0 {
		return fmt.Errorf("invalid user id: %d", id)
	}
	time.Sleep(100 * time.Millisecond)
	return nil
}

func main() {
	g, ctx := errgroup.WithContext(context.Background())

	userIDs := []int{1, 2, -1, 4} // -1 will cause an error

	for _, id := range userIDs {
		id := id // Capture loop variable
		g.Go(func() error {
			return fetchUser(id)
		})
	}

	// Wait for all goroutines and collect errors
	if err := g.Wait(); err != nil {
		fmt.Println("Error:", err)
		// Output: Error: invalid user id: -1
	}

	// Context is cancelled if any goroutine returns an error
	if ctx.Err() != nil {
		fmt.Println("Context cancelled:", ctx.Err())
	}
}
```

### Summary

Go's error handling philosophy emphasizes explicit error checking and clear error propagation. While it may seem verbose compared to exception-based systems, this approach makes error handling visible and forces you to think about failure cases at every step. Key takeaways:

- Always check and handle errors explicitly
- Wrap errors with context using `fmt.Errorf` and `%w`
- Use `errors.Is()` and `errors.As()` to inspect wrapped errors
- Create sentinel errors for well-defined, stable error conditions
- Follow idiomatic patterns like early returns and multiple return values
- Add context at each layer of your application
- Test error conditions thoroughly
- Don't log and return errors (choose one)
- Use custom error types when you need structured error information
- Reserve `panic` for truly exceptional situations

By following these practices, you'll write Go code that handles errors gracefully and provides clear diagnostic information when things go wrong.
