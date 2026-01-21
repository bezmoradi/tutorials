# Go > Introduction to The `log` Package

One of the differences between the `fmt.Println` function and `log.Println` is that the latter also prints the date and time:

```go
package main

import (
	"errors"
	"log"
)

func main() {
	err := errors.New("this is a custom-made error")
	log.Print(err) // 2026/01/21 18:40:38 this is a custom-made error
}
```

The `fmt` package formats an error value by calling its `Error` method. One other difference is that with the `log` package we can define **where** the log needs to go:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	f, err := os.Create("logs.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer f.Close()
	log.SetOutput(f)
	_, err = os.Open("NON_EXISTING_FILE")
	if err != nil {
		log.Println(err)
	}
}
```

In this program, we first create a brand-new file called `logs.txt`, then by calling `log.SetOutput(f)`, we instruct the `log` package to send logs to that file instead of `Stdout` (standard output, which is typically the terminal).

## Log Severity Levels

The `log` package provides different functions for different severity levels, each with distinct behavior.

### `log.Print()`, `log.Println()`, and `log.Printf()`

These are the standard logging functions that write to the configured output without any additional side effects:

```go
package main

import "log"

func main() {
	log.Print("This is a simple log message")
	log.Println("This message includes a newline")
	log.Printf("This is a %s message with value: %d", "formatted", 42)
}
```

### `log.Fatal()` and Its Variants

The `log.Fatal()` function is equivalent to calling `log.Print()` followed by `os.Exit(1)`. This means the program will terminate immediately after logging the message. Use this when encountering unrecoverable errors:

```go
package main

import (
	"log"
	"os"
)

func main() {
	f, err := os.Open("config.json")
	if err != nil {
		log.Fatal("Failed to open config file:", err)
		// Code after this line will never execute
	}
	defer f.Close()

	log.Println("This line won't be reached if the file doesn't exist")
}
```

The `log` package also provides `log.Fatalf()` for formatted output and `log.Fatalln()` for output with a newline. Keep in mind that because `Fatal` calls `os.Exit(1)`, deferred functions will not run. If you need cleanup code to execute, handle the error differently or ensure cleanup happens before calling `Fatal`.

### `log.Panic()` and Its Variants

The `log.Panic()` function is equivalent to calling `log.Print()` followed by `panic()`. Unlike `Fatal`, panics can be recovered using `defer` and `recover()`, making them useful for errors that might be handled higher up the call stack:

```go
package main

import (
	"fmt"
	"log"
)

func riskyOperation() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()

	// Simulate a critical error
	log.Panic("Something went terribly wrong!")
	fmt.Println("This line won't execute")
}

func main() {
	riskyOperation()
	fmt.Println("Program continues after recovery")
}
```

Similar to `Fatal`, the `log` package provides `log.Panicf()` and `log.Panicln()` variants. Use `Panic` when you want to allow potential recovery, and `Fatal` when the error is truly unrecoverable and the program must exit.

## Configuring Log Output Format

The `log` package allows you to customize the format of log messages using flags and prefixes.

### Using `log.SetFlags()`

The `log.SetFlags()` function controls what information appears in each log entry. You can combine multiple flags using the bitwise OR operator (`|`):

```go
package main

import (
	"log"
)

func main() {
	// Default behavior: date and time
	log.Println("Default format")

	// Show date only
	log.SetFlags(log.Ldate)
	log.Println("Date only")

	// Show time with microseconds
	log.SetFlags(log.Ltime | log.Lmicroseconds)
	log.Println("Time with microseconds")

	// Show full file path and line number
	log.SetFlags(log.Llongfile)
	log.Println("With full file path")

	// Show short file name and line number
	log.SetFlags(log.Lshortfile)
	log.Println("With short file name")

	// Combine multiple flags
	log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
	log.Println("Date, time, and file info")

	// Use UTC instead of local time
	log.SetFlags(log.Ldate | log.Ltime | log.LUTC)
	log.Println("UTC time")
}
```

Available flags include:

- `log.Ldate`: The date in the local time zone (2026/01/21)
- `log.Ltime`: The time in the local time zone (18:40:38)
- `log.Lmicroseconds`: Microsecond resolution (18:40:38.123456); assumes `Ltime` is set
- `log.Llongfile`: Full file name and line number (/path/to/file.go:42)
- `log.Lshortfile`: Final file name element and line number (file.go:42)
- `log.LUTC`: Use UTC rather than the local time zone
- `log.Lmsgprefix`: Move the prefix from the beginning of the line to before the message
- `log.LstdFlags`: Initial values for the standard logger (equivalent to `Ldate | Ltime`)

### Using `log.SetPrefix()`

The `log.SetPrefix()` function adds a prefix to every log message, which is useful for identifying the source of log messages in larger applications:

```go
package main

import "log"

func main() {
	// Set a prefix
	log.SetPrefix("[INFO] ")
	log.Println("Application started")

	// Change the prefix
	log.SetPrefix("[WARNING] ")
	log.Println("Low disk space")

	// Combine prefix with flags
	log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
	log.SetPrefix("[ERROR] ")
	log.Println("Failed to connect to database")
}
```

Output:
```text
[INFO] Application started
[WARNING] Low disk space
[ERROR] 2026/01/21 18:40:38 main.go:17: Failed to connect to database
```

By default, the prefix appears at the beginning of the line, before the timestamp. If you want the prefix to appear after the timestamp and before the message, use the `Lmsgprefix` flag:

```go
log.SetFlags(log.Ldate | log.Ltime | log.Lmsgprefix)
log.SetPrefix("[ERROR] ")
log.Println("Connection failed")
// Output: 2026/01/21 18:40:38 [ERROR] Connection failed
```

## Creating Custom Loggers

While the standard logger (accessed via `log.Println()`, `log.Fatal()`, etc.) is convenient, you often need multiple loggers with different configurations. The `log.New()` function creates custom logger instances.

### Basic Custom Logger

The `log.New()` function takes three parameters:
1. `out io.Writer`: Where the logger writes to
2. `prefix string`: Text to prepend to each log entry
3. `flag int`: Formatting flags (same as `SetFlags()`)

```go
package main

import (
	"log"
	"os"
)

func main() {
	// Create an info logger
	infoLogger := log.New(os.Stdout, "[INFO] ", log.Ldate|log.Ltime|log.Lshortfile)
	infoLogger.Println("Application started successfully")

	// Create an error logger
	errorLogger := log.New(os.Stderr, "[ERROR] ", log.Ldate|log.Ltime|log.Lshortfile)
	errorLogger.Println("Failed to connect to database")
}
```

### Multiple Loggers for Different Purposes

A common pattern is to create separate loggers for different severity levels or application components:

```go
package main

import (
	"io"
	"log"
	"os"
)

var (
	InfoLogger    *log.Logger
	WarningLogger *log.Logger
	ErrorLogger   *log.Logger
)

func init() {
	// Create or open a log file
	file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		log.Fatal("Failed to open log file:", err)
	}

	// Write to both file and stdout
	multiWriter := io.MultiWriter(file, os.Stdout)

	InfoLogger = log.New(multiWriter, "[INFO] ", log.Ldate|log.Ltime|log.Lshortfile)
	WarningLogger = log.New(multiWriter, "[WARNING] ", log.Ldate|log.Ltime|log.Lshortfile)
	ErrorLogger = log.New(multiWriter, "[ERROR] ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
	InfoLogger.Println("Starting application")
	WarningLogger.Println("Configuration file not found, using defaults")
	ErrorLogger.Println("Failed to connect to external API")
}
```

In this example, we use `io.MultiWriter` to send logs to both a file and stdout simultaneously. This is useful for production applications where you want logs persisted to disk while still being visible in the console.

### Component-Specific Loggers

For larger applications, you might want separate loggers for different components:

```go
package main

import (
	"log"
	"os"
)

type Database struct {
	logger *log.Logger
}

func NewDatabase() *Database {
	return &Database{
		logger: log.New(os.Stdout, "[DATABASE] ", log.Ldate|log.Ltime),
	}
}

func (db *Database) Connect() {
	db.logger.Println("Attempting to connect")
	// Connection logic here
	db.logger.Println("Connected successfully")
}

type APIServer struct {
	logger *log.Logger
}

func NewAPIServer() *APIServer {
	return &APIServer{
		logger: log.New(os.Stdout, "[API] ", log.Ldate|log.Ltime),
	}
}

func (api *APIServer) Start() {
	api.logger.Println("Starting server on port 8080")
	// Server startup logic
}

func main() {
	db := NewDatabase()
	db.Connect()

	api := NewAPIServer()
	api.Start()
}
```

This approach makes it easy to identify which component generated each log message, especially helpful when debugging complex applications.

## Thread Safety and Performance Considerations

One of the key advantages of the `log` package is its built-in thread safety. All loggers created with `log.New()` and the standard logger are safe for concurrent use by multiple goroutines.

### Thread Safety

The `log` package uses mutex locks internally to synchronize access when writing log entries. This means you can safely call logging functions from multiple goroutines without additional synchronization:

```go
package main

import (
	"log"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 0; i < 5; i++ {
		// Safe to call from multiple goroutines
		log.Printf("Worker %d: processing item %d", id, i)
	}
}

func main() {
	var wg sync.WaitGroup

	// Launch multiple goroutines
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}

	wg.Wait()
}
```

The output from different goroutines will be properly interleaved without corruption, though the order is non-deterministic.

### Performance Implications

While thread safety is essential, the mutex locking mechanism does add some overhead. For most applications this overhead is negligible, but in scenarios with extremely high concurrency and logging frequency, it could become a bottleneck.

Performance optimization strategies:

1. **Reduce log frequency**: Only log meaningful events, not every operation
2. **Use appropriate log levels**: Don't log debug information in production
3. **Buffered writers**: Use `bufio.Writer` to batch writes and reduce I/O operations
4. **Asynchronous logging**: For high-performance applications, consider buffering logs in memory and writing them asynchronously

Example with buffered writer:

```go
package main

import (
	"bufio"
	"log"
	"os"
)

func main() {
	file, err := os.Create("buffered.log")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	// Create a buffered writer
	writer := bufio.NewWriter(file)
	defer writer.Flush() // Important: flush before program exits

	logger := log.New(writer, "[APP] ", log.Ldate|log.Ltime)

	// These writes are buffered
	for i := 0; i < 1000; i++ {
		logger.Printf("Log entry %d", i)
	}

	// Flush ensures all buffered data is written
	writer.Flush()
}
```

## When to Use `log` vs `fmt`

Understanding when to use each package is crucial for writing maintainable Go applications.

### Use `log` When:

1. **You need timestamps**: The `log` package automatically includes date and time information
2. **Logging to files or multiple outputs**: `log` makes it easy to redirect output
3. **Production applications**: The structured format and thread safety make `log` suitable for production
4. **You need different severity levels**: Use `Fatal` for unrecoverable errors and `Panic` for critical issues
5. **Concurrent operations**: Thread safety is built-in
6. **You want consistent formatting**: Prefixes and flags provide uniform log format across your application

### Use `fmt` When:

1. **Simple output to console**: For development, debugging, or command-line tools
2. **User-facing messages**: Output that end users will see
3. **Formatted strings without logging metadata**: When you don't need timestamps or file information
4. **Maximum performance**: `fmt` has slightly less overhead since it doesn't include locking or formatting
5. **Fine-grained control**: When you need complete control over output format

### Example Comparison:

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	// fmt: Simple output, no timestamp
	fmt.Println("User logged in successfully")

	// log: Includes timestamp and can be redirected
	log.Println("User logged in successfully")

	// fmt: Best for user-facing messages
	fmt.Printf("Welcome, %s!\n", "Alice")

	// log: Best for internal application events
	log.Printf("Processing request from user: %s", "Alice")
}
```

Output:
```text
User logged in successfully
2026/01/21 18:40:38 User logged in successfully
Welcome, Alice!
2026/01/21 18:40:38 Processing request from user: Alice
```

### Best Practices

1. **Separate user output from logs**: Use `fmt` for user-facing messages and `log` for internal application events
2. **Create logger instances**: Instead of using the standard logger everywhere, create specific logger instances for different components
3. **Don't log sensitive information**: Avoid logging passwords, API keys, or personal data
4. **Use appropriate severity**: Reserve `Fatal` for truly unrecoverable errors and use `Println` for informational messages
5. **Include context**: Add relevant context (user ID, request ID, etc.) to make logs useful for debugging
6. **Consider structured logging**: For complex applications, consider third-party libraries like `zap` or `zerolog` that provide structured logging with better performance
7. **Flush buffered writers**: Always flush buffered writers before program exit to ensure all logs are written

The standard `log` package is excellent for small to medium applications. For large-scale production systems with high-performance requirements, consider exploring structured logging libraries built on top of the standard library's foundation.
