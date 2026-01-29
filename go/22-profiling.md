# Go > Profiling

Profiling is the process of measuring your program's performance to identify bottlenecks and optimization opportunities. Go provides powerful built-in profiling tools through the `runtime/pprof` and `net/http/pprof` packages, along with the `pprof` visualization tool. This tutorial covers how to profile your Go applications effectively to understand where time and memory are being spent.

This tutorial covers profiling features in Go 1.25 and later, including the new trace flight recorder and goroutine leak detection capabilities.

## Why Profile?

Before optimizing code, you need data. Profiling helps you:

1. **Identify hotspots**: Find which functions consume the most CPU time
2. **Detect memory leaks**: Track memory allocation and identify unnecessary allocations
3. **Understand concurrency issues**: Find goroutine blocking and mutex contention
4. **Make informed decisions**: Optimize based on data, not assumptions
5. **Prevent regressions**: Establish performance baselines and track changes over time

Donald Knuth famously said, "Premature optimization is the root of all evil". Profiling ensures you optimize the right parts of your code.

## Types of Profiling

Go supports several types of profiling:

1. **CPU profiling**: Shows where your program spends CPU time
2. **Memory (heap) profiling**: Tracks memory allocations
3. **Goroutine profiling**: Shows all currently running goroutines
4. **Goroutine leak profiling** (Go 1.26+): Automatically detects goroutines that are blocked and unreachable
5. **Block profiling**: Identifies where goroutines block waiting on synchronization primitives
6. **Mutex profiling**: Reports lock contention
7. **Thread creation profiling**: Tracks OS thread creation

Each profile type helps diagnose different performance issues. Let's explore each in detail.

## CPU Profiling

CPU profiling determines where your program spends time while actively consuming CPU cycles (as opposed to sleeping or waiting for I/O). This is typically the first profiling type you'll use when investigating performance issues.

### Basic CPU Profiling with `runtime/pprof`

Here's a simple example of CPU profiling:

```go
package main

import (
	"fmt"
	"os"
	"runtime/pprof"
	"time"
)

func slowFunction() {
	// Simulate CPU-intensive work
	total := 0
	for i := 0; i < 1000000000; i++ {
		total += i
	}
}

func fasterFunction() {
	// Less intensive work
	total := 0
	for i := 0; i < 100000000; i++ {
		total += i
	}
}

func main() {
	// Create CPU profile file
	f, err := os.Create("cpu.prof")
	if err != nil {
		fmt.Println("Could not create CPU profile:", err)
		return
	}
	defer f.Close()

	// Start CPU profiling
	if err := pprof.StartCPUProfile(f); err != nil {
		fmt.Println("Could not start CPU profile:", err)
		return
	}
	defer pprof.StopCPUProfile()

	// Run your program
	start := time.Now()
	slowFunction()
	fasterFunction()
	fmt.Printf("Execution time: %v\n", time.Since(start))
}
```

After running this program, you'll have a `cpu.prof` file. Analyze it with:

```bash
go tool pprof cpu.prof
```

This opens an interactive shell where you can explore the profile data.

### Understanding CPU Profile Output

Common `pprof` commands:

```text
(pprof) top
Shows the functions consuming the most CPU time

(pprof) top10
Shows the top 10 functions

(pprof) list slowFunction
Shows the source code of slowFunction with time annotations

(pprof) web
Opens a graphical visualization in your browser (requires Graphviz)

(pprof) pdf
Generates a PDF of the call graph
```

Example output from `top`:

```text
Showing nodes accounting for 2.51s, 98.43% of 2.55s total
      flat  flat%   sum%        cum   cum%
     1.67s 65.49% 65.49%      1.67s 65.49%  main.slowFunction
     0.84s 32.94% 98.43%      0.84s 32.94%  main.fasterFunction
```

-   **flat**: Time spent in the function itself
-   **cum** (cumulative): Time spent in the function and its callees

### CPU Profiling with Command-Line Flags

You can also enable CPU profiling via command-line flags using the `flag` package:

```go
package main

import (
	"flag"
	"fmt"
	"os"
	"runtime/pprof"
)

var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")

func doWork() {
	total := 0
	for i := 0; i < 1000000000; i++ {
		total += i
	}
	fmt.Println("Work completed")
}

func main() {
	flag.Parse()

	if *cpuprofile != "" {
		f, err := os.Create(*cpuprofile)
		if err != nil {
			fmt.Println("Could not create CPU profile:", err)
			return
		}
		defer f.Close()

		if err := pprof.StartCPUProfile(f); err != nil {
			fmt.Println("Could not start CPU profile:", err)
			return
		}
		defer pprof.StopCPUProfile()
	}

	doWork()
}
```

Run with:

```bash
go run main.go -cpuprofile=cpu.prof
```

### When to Use CPU Profiling

CPU profiling is useful when:

-   Your application is running slower than expected
-   You need to identify hot paths in your code
-   You want to verify that optimizations actually improved performance
-   CPU usage is high and you need to find the cause

## Memory Profiling

Memory profiling helps you understand your program's memory allocation patterns. The heap profile tracks both live objects currently in memory and all objects allocated since program start.

### Basic Memory Profiling

Unlike CPU profiling which requires `Start` and `Stop` functions, memory profiling is a snapshot taken at a specific point:

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/pprof"
)

func allocateMemory() [][]byte {
	// Allocate some memory
	var data [][]byte
	for i := 0; i < 1000; i++ {
		// Each iteration allocates 1MB
		data = append(data, make([]byte, 1024*1024))
	}
	return data
}

func main() {
	// Allocate memory
	data := allocateMemory()

	// Force garbage collection to get accurate results
	runtime.GC()

	// Create memory profile file
	f, err := os.Create("mem.prof")
	if err != nil {
		fmt.Println("Could not create memory profile:", err)
		return
	}
	defer f.Close()

	// Write heap profile
	if err := pprof.WriteHeapProfile(f); err != nil {
		fmt.Println("Could not write memory profile:", err)
		return
	}

	fmt.Printf("Allocated %d MB\n", len(data))
}
```

Analyze the memory profile:

```bash
go tool pprof mem.prof
```

### Memory Profile Modes

The `pprof` tool provides four different views of memory profiles:

1. **-inuse_space** (default): Shows memory currently in use, scaled by size
2. **-inuse_objects**: Shows memory currently in use, scaled by object count
3. **-alloc_space**: Shows all allocated memory since program start, by size
4. **-alloc_objects**: Shows all allocated memory since program start, by count

Example usage:

```bash
# See currently allocated memory by size
go tool pprof -inuse_space mem.prof

# See all allocations since start
go tool pprof -alloc_space mem.prof

# See allocation counts
go tool pprof -alloc_objects mem.prof
```

### Finding Memory Leaks

To find memory leaks, take two memory profiles at different times and compare them:

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/pprof"
	"time"
)

var leakySlice [][]byte

func leakyFunction() {
	// This function "leaks" memory by holding references
	for i := 0; i < 100; i++ {
		leakySlice = append(leakySlice, make([]byte, 1024*1024))
	}
}

func main() {
	// First profile
	runtime.GC()
	f1, _ := os.Create("mem-before.prof")
	pprof.WriteHeapProfile(f1)
	f1.Close()

	// Run potentially leaky code
	for i := 0; i < 10; i++ {
		leakyFunction()
		time.Sleep(100 * time.Millisecond)
	}

	// Second profile
	runtime.GC()
	f2, _ := os.Create("mem-after.prof")
	pprof.WriteHeapProfile(f2)
	f2.Close()

	fmt.Println("Memory profiles created")
}
```

Compare the profiles:

```bash
go tool pprof -base mem-before.prof mem-after.prof
```

This shows the difference in memory allocations between the two snapshots, making leaks easier to spot.

### Profiling Specific Allocations

You can focus on specific types of allocations:

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/pprof"
)

type LargeStruct struct {
	Data [1024]int
	Name string
}

func createStructs() []*LargeStruct {
	var structs []*LargeStruct
	for i := 0; i < 10000; i++ {
		structs = append(structs, &LargeStruct{
			Name: fmt.Sprintf("Struct %d", i),
		})
	}
	return structs
}

func createSlices() [][]int {
	var slices [][]int
	for i := 0; i < 5000; i++ {
		slices = append(slices, make([]int, 100))
	}
	return slices
}

func main() {
	structs := createStructs()
	slices := createSlices()

	runtime.GC()

	f, _ := os.Create("mem-detailed.prof")
	defer f.Close()
	pprof.WriteHeapProfile(f)

	fmt.Printf("Created %d structs and %d slices\n", len(structs), len(slices))
}
```

When analyzing with `pprof`, use the `list` command to see line-by-line allocations:

```text
(pprof) list createStructs
```

### When to Use Memory Profiling

Memory profiling is useful when:

-   Your application's memory usage grows unexpectedly
-   You suspect a memory leak
-   You need to reduce memory allocations for performance
-   You want to understand the memory footprint of your data structures
-   Out-of-memory errors occur in production

## HTTP Profiling with `net/http/pprof`

For long-running services like web servers, the `net/http/pprof` package provides an HTTP interface to runtime profiling data. This is especially useful for production systems where you can't easily stop and start the program.

### Setting Up HTTP Profiling

Simply import the package and it automatically registers handlers with the default HTTP mux:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	_ "net/http/pprof" // Blank import registers pprof handlers
	"time"
)

func heavyHandler(w http.ResponseWriter, r *http.Request) {
	// Simulate CPU-intensive work
	total := 0
	for i := 0; i < 100000000; i++ {
		total += i
	}
	fmt.Fprintf(w, "Result: %d", total)
}

func memoryHandler(w http.ResponseWriter, r *http.Request) {
	// Simulate memory allocation
	data := make([]byte, 10*1024*1024) // 10 MB
	data[0] = 1
	fmt.Fprintf(w, "Allocated %d bytes", len(data))
}

func main() {
	http.HandleFunc("/heavy", heavyHandler)
	http.HandleFunc("/memory", memoryHandler)

	fmt.Println("Server starting on :8080")
	fmt.Println("Profiling available at http://localhost:8080/debug/pprof/")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Accessing Profile Data

With the server running, you can access various profiles:

**Via web browser:**

-   http://localhost:8080/debug/pprof/ - Index page listing all profiles
-   http://localhost:8080/debug/pprof/heap - Heap profile
-   http://localhost:8080/debug/pprof/goroutine - Goroutine profile
-   http://localhost:8080/debug/pprof/goroutineleak - Goroutine leak profile (Go 1.26+ with GOEXPERIMENT=goroutineleakprofile)
-   http://localhost:8080/debug/pprof/threadcreate - Thread creation profile
-   http://localhost:8080/debug/pprof/block - Block profile
-   http://localhost:8080/debug/pprof/mutex - Mutex profile

**Via `go tool pprof` directly:**

```bash
# CPU profile (30 seconds by default)
go tool pprof http://localhost:8080/debug/pprof/profile

# CPU profile for 60 seconds
go tool pprof http://localhost:8080/debug/pprof/profile?seconds=60

# Heap profile
go tool pprof http://localhost:8080/debug/pprof/heap

# Goroutine profile
go tool pprof http://localhost:8080/debug/pprof/goroutine
```

### Custom HTTP Mux

If you're using a custom HTTP mux (like with `gorilla/mux`), you need to register the handlers manually:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"net/http/pprof"

	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	// Register application routes
	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Hello, World!")
	})

	// Register pprof handlers
	r.HandleFunc("/debug/pprof/", pprof.Index)
	r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
	r.HandleFunc("/debug/pprof/profile", pprof.Profile)
	r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
	r.HandleFunc("/debug/pprof/trace", pprof.Trace)

	// Manually register other pprof handlers
	r.Handle("/debug/pprof/goroutine", pprof.Handler("goroutine"))
	r.Handle("/debug/pprof/goroutineleak", pprof.Handler("goroutineleak")) // Go 1.26+
	r.Handle("/debug/pprof/heap", pprof.Handler("heap"))
	r.Handle("/debug/pprof/threadcreate", pprof.Handler("threadcreate"))
	r.Handle("/debug/pprof/block", pprof.Handler("block"))
	r.Handle("/debug/pprof/mutex", pprof.Handler("mutex"))

	fmt.Println("Server with custom mux on :8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}
```

### Securing pprof Endpoints

Never expose pprof endpoints to the public internet in production. Use one of these approaches:

**1. Separate port for profiling:**

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	_ "net/http/pprof"
)

func main() {
	// Production endpoints
	go func() {
		http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			fmt.Fprint(w, "Production API")
		})
		log.Fatal(http.ListenAndServe(":8080", nil))
	}()

	// Profiling endpoints on separate port
	// Only accessible internally
	fmt.Println("Profiling on :6060 (internal only)")
	log.Fatal(http.ListenAndServe("localhost:6060", nil))
}
```

**2. Authentication middleware:**

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	_ "net/http/pprof"
)

func authMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Simple token-based auth (use proper auth in production)
		token := r.Header.Get("X-Debug-Token")
		if token != "secret-debug-token" {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}
		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()

	// Application routes
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Public API")
	})

	// Protected debug routes
	debugMux := http.NewServeMux()
	debugMux.Handle("/debug/", http.DefaultServeMux)

	mux.Handle("/debug/", authMiddleware(debugMux))

	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

### Real-Time Profiling Example

Here's a complete example showing how to profile a live service:

```go
package main

import (
	"fmt"
	"log"
	"math/rand"
	"net/http"
	_ "net/http/pprof"
	"sync"
	"time"
)

var cache = make(map[string]string)
var cacheMutex sync.RWMutex

func cacheHandler(w http.ResponseWriter, r *http.Request) {
	key := r.URL.Query().Get("key")

	cacheMutex.RLock()
	value, exists := cache[key]
	cacheMutex.RUnlock()

	if exists {
		fmt.Fprintf(w, "Cached value: %s", value)
		return
	}

	// Simulate expensive operation
	time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
	value = fmt.Sprintf("value-%d", rand.Int())

	cacheMutex.Lock()
	cache[key] = value
	cacheMutex.Unlock()

	fmt.Fprintf(w, "New value: %s", value)
}

func main() {
	http.HandleFunc("/cache", cacheHandler)

	fmt.Println("Server running on :8080")
	fmt.Println("Try: curl 'http://localhost:8080/cache?key=test'")
	fmt.Println("Profile: go tool pprof http://localhost:8080/debug/pprof/profile")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

While the server runs and handles requests, profile it:

```bash
# Profile for 30 seconds while making requests
go tool pprof http://localhost:8080/debug/pprof/profile
```

## Goroutine, Block, and Mutex Profiling

These specialized profiles help diagnose concurrency issues in Go programs.

### Goroutine Profiling

Goroutine profiling shows all currently running goroutines and their call stacks. This helps identify goroutine leaks or understand concurrency patterns.

```go
package main

import (
	"fmt"
	"os"
	"runtime/pprof"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	// Simulate work
	time.Sleep(5 * time.Second)
	fmt.Printf("Worker %d done\n", id)
}

func leakyGoroutine() {
	// This goroutine never finishes
	for {
		time.Sleep(1 * time.Second)
	}
}

func main() {
	var wg sync.WaitGroup

	// Start some workers
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}

	// Start a leaky goroutine
	go leakyGoroutine()

	// Take goroutine profile
	time.Sleep(1 * time.Second)
	f, _ := os.Create("goroutine.prof")
	defer f.Close()
	pprof.Lookup("goroutine").WriteTo(f, 0)

	wg.Wait()
	fmt.Println("All workers done")
}
```

Analyze the goroutine profile:

```bash
go tool pprof goroutine.prof
```

In the pprof shell, use `top` to see where goroutines are blocked:

```text
(pprof) top
(pprof) list leakyGoroutine
```

### Goroutine Leak Detection (Go 1.26+)

Go 1.26 introduces an experimental goroutine leak profile that automatically identifies goroutines that are blocked and unreachable from any runnable goroutine. This helps detect goroutine leaks without manual analysis.

To enable the goroutine leak profile, build your program with the experiment flag:

```bash
GOEXPERIMENT=goroutineleakprofile go build -o myapp main.go
```

The leak profile has zero runtime overhead unless actively collected, making it safe for production use.

#### Using the Goroutine Leak Profile

```go
package main

import (
	"fmt"
	"net/http"
	_ "net/http/pprof"
	"time"
)

func leakyWorker(ch chan struct{}) {
	// This goroutine will leak because nothing sends on ch
	<-ch
	fmt.Println("This never executes")
}

func main() {
	// Start multiple leaky goroutines
	for i := 0; i < 10; i++ {
		ch := make(chan struct{})
		go leakyWorker(ch)
		// Channel reference is lost, goroutine will block forever
	}

	// Start HTTP server with pprof
	fmt.Println("Server on :8080")
	fmt.Println("Leak profile: go tool pprof http://localhost:8080/debug/pprof/goroutineleak")
	http.ListenAndServe(":8080", nil)
}
```

Access the leak profile:

```bash
# Via pprof tool (after building with GOEXPERIMENT=goroutineleakprofile)
go tool pprof http://localhost:8080/debug/pprof/goroutineleak

# Or save to file
curl http://localhost:8080/debug/pprof/goroutineleak > leak.prof
go tool pprof leak.prof
```

The leak profile shows only goroutines that are both:
1. Blocked on a concurrency primitive (channel, mutex, etc.)
2. Unreachable from any runnable goroutine

This filters out intentionally-blocked goroutines (like HTTP server workers waiting for connections) and focuses on actual leaks.

**Note**: This feature is experimental in Go 1.26 and planned to become the default in Go 1.27. The API may change based on feedback.

### Block Profiling

Block profiling shows where goroutines block waiting on synchronization primitives (channels, mutexes, select statements). This is not enabled by default.

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/pprof"
	"sync"
	"time"
)

var mu sync.Mutex

func contentionFunction(id int) {
	for i := 0; i < 5; i++ {
		mu.Lock()
		// Hold the lock for a while
		time.Sleep(100 * time.Millisecond)
		fmt.Printf("Goroutine %d: iteration %d\n", id, i)
		mu.Unlock()
	}
}

func main() {
	// Enable block profiling
	// 1 = record every blocking event (most accurate, but has overhead)
	runtime.SetBlockProfileRate(1)

	var wg sync.WaitGroup

	// Start multiple goroutines that will contend for the lock
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			contentionFunction(id)
		}(i)
	}

	wg.Wait()

	// Write block profile
	f, _ := os.Create("block.prof")
	defer f.Close()
	pprof.Lookup("block").WriteTo(f, 0)

	fmt.Println("Block profile created")
}
```

Analyze the block profile:

```bash
go tool pprof block.prof
```

Block profiling shows time spent blocked, not CPU time, so it complements CPU profiling by revealing where your program is waiting.

### Mutex Profiling

Mutex profiling reports lock contention. It tracks how much time goroutines spend waiting to acquire mutexes.

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/pprof"
	"sync"
)

type Counter struct {
	mu    sync.Mutex
	value int
}

func (c *Counter) Increment() {
	c.mu.Lock()
	// Simulate some work while holding the lock
	total := 0
	for i := 0; i < 10000000; i++ {
		total += i
	}
	c.value++
	c.mu.Unlock()
}

func main() {
	// Enable mutex profiling
	// 1 = sample 100% of mutex contention events
	// Higher values sample less frequently (e.g., 10 = sample 10%)
	runtime.SetMutexProfileFraction(1)

	counter := &Counter{}
	var wg sync.WaitGroup

	// Create contention by having multiple goroutines increment
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for j := 0; j < 100; j++ {
				counter.Increment()
			}
		}()
	}

	wg.Wait()

	// Write mutex profile
	f, _ := os.Create("mutex.prof")
	defer f.Close()
	pprof.Lookup("mutex").WriteTo(f, 0)

	fmt.Printf("Final count: %d\n", counter.value)
	fmt.Println("Mutex profile created")
}
```

Analyze the mutex profile:

```bash
go tool pprof mutex.prof
```

The mutex profile shows cumulative time spent waiting for locks. Stack traces point to the **unlock** operations that caused contention (not the lock acquisitions).

### HTTP Endpoint for Concurrency Profiles

These profiles are also available via HTTP when using `net/http/pprof`:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	_ "net/http/pprof"
	"runtime"
	"sync"
	"time"
)

func init() {
	// Enable block and mutex profiling
	runtime.SetBlockProfileRate(1)
	runtime.SetMutexProfileFraction(1)
}

var mu sync.Mutex

func slowHandler(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	time.Sleep(100 * time.Millisecond)
	mu.Unlock()
	fmt.Fprint(w, "Done")
}

func main() {
	http.HandleFunc("/slow", slowHandler)

	fmt.Println("Server on :8080")
	fmt.Println("Goroutines: http://localhost:8080/debug/pprof/goroutine")
	fmt.Println("Block: http://localhost:8080/debug/pprof/block")
	fmt.Println("Mutex: http://localhost:8080/debug/pprof/mutex")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Access the profiles:

```bash
# Goroutine profile
go tool pprof http://localhost:8080/debug/pprof/goroutine

# Goroutine leak profile (Go 1.26+ with GOEXPERIMENT=goroutineleakprofile)
go tool pprof http://localhost:8080/debug/pprof/goroutineleak

# Block profile
go tool pprof http://localhost:8080/debug/pprof/block

# Mutex profile
go tool pprof http://localhost:8080/debug/pprof/mutex
```

### Interpreting Concurrency Profiles

**Goroutine profile:**

-   High goroutine count might indicate leaks
-   Look for goroutines stuck in the same state
-   Check if goroutines are properly cleaned up

**Block profile:**

-   Shows time waiting on channels, select, mutexes
-   High block time indicates synchronization bottlenecks
-   Consider redesigning to reduce blocking

**Mutex profile:**

-   Shows lock contention hotspots
-   High contention suggests lock is held too long or too frequently
-   Consider lock-free data structures, finer-grained locking, or reducing critical section size

### When to Use These Profiles

-   **Goroutine profiling**: When goroutine count grows unexpectedly or you suspect leaks
-   **Goroutine leak profiling** (Go 1.26+): Automatic detection of unreachable blocked goroutines without manual analysis
-   **Block profiling**: When CPU usage is low but throughput is also low
-   **Mutex profiling**: When you suspect lock contention is affecting performance

## Using `go tool pprof`

The `pprof` tool provides both interactive and non-interactive ways to analyze profiles.

### Interactive Mode

Launch `pprof` in interactive mode:

```bash
go tool pprof cpu.prof
```

Common interactive commands:

```text
top [N]          Show top N entries (default 10)
top -cum [N]     Show top N by cumulative time
list <func>      Show source code with annotations for function
web              Open call graph in browser (requires Graphviz)
pdf              Generate PDF call graph
png              Generate PNG call graph
peek <regex>     Show callers and callees of functions matching regex
traces           Print stack traces
help             Show all commands
```

### Non-Interactive Mode

Generate reports without entering interactive mode:

```bash
# Generate text report
go tool pprof -text cpu.prof

# Generate top 20 functions
go tool pprof -top cpu.prof | head -20

# Generate call graph
go tool pprof -pdf cpu.prof > profile.pdf

# Compare two profiles
go tool pprof -base old.prof new.prof

# Focus on specific function
go tool pprof -focus=myFunction cpu.prof

# Ignore specific functions
go tool pprof -ignore=runtime cpu.prof
```

### Web UI

`pprof` includes a powerful web UI:

```bash
go tool pprof -http=:8080 cpu.prof
```

This opens a browser with interactive visualization. In Go 1.26+, the web UI defaults to flame graph view (previous versions defaulted to the call graph view). You can switch between views using the "View" menu: flame graph, graph, peek, source, disassembly, and top.

### Flame Graphs

Flame graphs visualize where time is spent in a hierarchical way:

```bash
# Generate flame graph (requires -http flag)
go tool pprof -http=:8080 cpu.prof
```

In Go 1.26+, the web UI opens directly to the flame graph view. In earlier versions, click "VIEW" → "Flame Graph" to see the visualization.

### Comparing Profiles

Compare before and after optimization:

```bash
# Create baseline
go test -cpuprofile=old.prof -bench=.

# Make changes, then profile again
go test -cpuprofile=new.prof -bench=.

# Compare
go tool pprof -base=old.prof new.prof
```

This shows the delta between profiles, highlighting improvements or regressions.

## Execution Tracing

Go's execution tracer provides a different view than profiling. While profiling shows where time is spent, tracing shows **what happened over time**: goroutine scheduling, syscalls, GC events, and more.

### Creating a Trace

```go
package main

import (
	"fmt"
	"os"
	"runtime/trace"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	total := 0
	for i := 0; i < 100000000; i++ {
		total += i
	}
	fmt.Printf("Worker %d: %d\n", id, total)
}

func main() {
	// Create trace file
	f, err := os.Create("trace.out")
	if err != nil {
		panic(err)
	}
	defer f.Close()

	// Start tracing
	if err := trace.Start(f); err != nil {
		panic(err)
	}
	defer trace.Stop()

	// Run program
	var wg sync.WaitGroup
	for i := 0; i < 4; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}
	wg.Wait()
}
```

View the trace:

```bash
go tool trace trace.out
```

This opens a web UI showing:

-   Goroutine analysis
-   Network blocking profile
-   Synchronization blocking profile
-   Syscall blocking profile
-   Scheduler latency profile
-   Timeline view of all goroutines

### When to Use Tracing vs Profiling vs Flight Recording

**Use profiling when:**

-   You want to know where time is spent
-   You need to optimize CPU or memory usage
-   You want quantitative data (percentages, totals)

**Use flight recording when:**

-   You need production-safe continuous tracing
-   You want to capture rare events retroactively
-   Full tracing overhead is unacceptable
-   You need the events leading up to an error

**Use full tracing when:**

-   You need to understand timing and causality in detail
-   You want to see goroutine execution patterns
-   You need to debug complex concurrency issues
-   You want to see GC impact with complete history
-   CPU profiling shows normal results but performance is still poor
-   You can reproduce the issue and accept the overhead

### Trace Caveats

Tracing has significant overhead (can slow your program by 2-10x) and generates large files, so use it carefully in production. Traces are best for short runs (seconds to minutes) focused on specific scenarios.

## Flight Recorder (Go 1.25+)

Go 1.25 introduced the trace flight recorder, a lightweight continuous tracing mechanism that records execution traces into an in-memory ring buffer with minimal overhead. Unlike full execution tracing, the flight recorder is practical for production use and can capture rare events without the performance cost of always-on tracing.

### How Flight Recording Works

The flight recorder continuously captures trace data in a circular buffer. When you need to investigate an issue, you can snapshot the recent trace history without having planned ahead. This is similar to airplane black boxes that continuously record flight data.

Key benefits:
- Minimal performance overhead compared to full tracing
- Always-on capability for production systems
- Captures recent history leading up to an event
- On-demand snapshots via `WriteTo()`

### Basic Flight Recording Example

```go
package main

import (
	"fmt"
	"os"
	"runtime/trace"
	"time"
)

func simulateWork() {
	total := 0
	for i := 0; i < 50000000; i++ {
		total += i
	}
}

func main() {
	// Start flight recording
	if err := trace.StartFlightRecording(); err != nil {
		panic(err)
	}
	defer trace.StopFlightRecording()

	// Do work
	fmt.Println("Starting work with flight recording enabled...")
	for i := 0; i < 5; i++ {
		simulateWork()
		time.Sleep(100 * time.Millisecond)
	}

	// Something interesting happened, capture the trace
	fmt.Println("Capturing flight recording snapshot...")
	f, err := os.Create("flight-trace.out")
	if err != nil {
		panic(err)
	}
	defer f.Close()

	// Snapshot the flight recorder buffer
	if err := trace.FlightRecorder.WriteTo(f); err != nil {
		panic(err)
	}

	fmt.Println("Flight recording saved to flight-trace.out")
}
```

View the captured trace:

```bash
go tool trace flight-trace.out
```

### Flight Recording in HTTP Services

Flight recording is especially useful for long-running services where you want to debug rare issues:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"runtime/trace"
	"time"
)

var requestCount int

func handleRequest(w http.ResponseWriter, r *http.Request) {
	requestCount++

	// Simulate occasional slow request
	if requestCount%10 == 0 {
		time.Sleep(500 * time.Millisecond)
	}

	fmt.Fprintf(w, "Request #%d processed\n", requestCount)
}

func handleSnapshot(w http.ResponseWriter, r *http.Request) {
	// Capture flight recording snapshot on demand
	w.Header().Set("Content-Type", "application/octet-stream")
	w.Header().Set("Content-Disposition", "attachment; filename=flight-trace.out")

	if err := trace.FlightRecorder.WriteTo(w); err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
}

func main() {
	// Enable flight recording
	if err := trace.StartFlightRecording(); err != nil {
		log.Fatal(err)
	}
	defer trace.StopFlightRecording()

	http.HandleFunc("/", handleRequest)
	http.HandleFunc("/debug/flight-trace", handleSnapshot)

	fmt.Println("Server on :8080")
	fmt.Println("Generate load: curl http://localhost:8080/")
	fmt.Println("Capture trace: curl http://localhost:8080/debug/flight-trace > trace.out")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

When you notice an issue (slow request, error, etc.), hit the `/debug/flight-trace` endpoint to capture recent trace history.

### Flight Recording Configuration

You can configure the flight recorder buffer size (default is typically sufficient for most use cases):

```go
// The flight recorder uses a default buffer size
// Check runtime/trace documentation for current defaults and configuration options
```

### Flight Recording vs Full Tracing

| Feature | Flight Recording | Full Tracing |
|---------|-----------------|--------------|
| Overhead | Low (1-5%) | High (2-10x slowdown) |
| Production use | Yes | Risky |
| Data capture | Continuous ring buffer | Everything from start to stop |
| File size | Fixed buffer size | Grows unbounded |
| When to use | Always-on debugging | Detailed analysis of specific scenarios |

### When to Use Flight Recording

Flight recording is ideal when:
- You need to debug rare production issues
- You want always-on tracing capability without performance cost
- You need to investigate problems retroactively
- Full tracing overhead is unacceptable
- You want to capture the events leading up to an error

Use full tracing when you need complete execution history for detailed analysis of a specific, reproducible scenario.

## Best Practices

### 1. Profile Before Optimizing

Never optimize without profiling first. Developers are notoriously bad at guessing bottlenecks.

### 2. Profile in Realistic Conditions

Profile with production-like data, load, and environment. Synthetic benchmarks can miss real-world issues.

### 3. Use the Right Profile Type

-   Slow program → CPU profile
-   High memory usage → Heap profile
-   Low CPU but poor throughput → Block/mutex profile
-   Goroutine count growing → Goroutine profile or goroutine leak profile (Go 1.26+)
-   Suspected goroutine leaks → Goroutine leak profile (Go 1.26+)
-   Weird timing issues → Flight recording (Go 1.25+) or full trace
-   Rare production issues → Flight recording (Go 1.25+)

### 4. Focus on the Biggest Wins

Optimize functions at the top of the profile first. Optimizing a function consuming 1% of time won't noticeably improve performance.

### 5. Profile One Thing at a Time

Don't run multiple profile types simultaneously. CPU profiling can skew memory profiling and vice versa.

### 6. Use Benchmarks with Profiling

Combine profiling with benchmarks to measure optimization impact:

```go
func BenchmarkMyFunction(b *testing.B) {
	for i := 0; i < b.N; i++ {
		myFunction()
	}
}
```

Run with profiling:

```bash
go test -bench=. -cpuprofile=cpu.prof
go test -bench=. -memprofile=mem.prof
```

### 7. Establish Baselines

Profile regularly and track metrics over time. This helps catch performance regressions early.

### 8. Be Careful in Production

-   Don't leave profiling always enabled (overhead)
-   Secure pprof endpoints (never expose publicly)
-   Use separate ports or authentication for debug endpoints
-   Monitor profile collection overhead

### 9. Consider Continuous Profiling

For critical services, consider continuous profiling tools that safely collect profiles in production (like Google Cloud Profiler or Datadog Continuous Profiler).

### 10. Document Your Findings

When you optimize based on profiles, document:

-   What the profile showed
-   What you changed
-   The measured improvement
-   Any tradeoffs made

This creates institutional knowledge and helps future debugging.

## Summary

Go's profiling tools provide deep insights into program behavior:

-   **CPU profiling** identifies hot code paths
-   **Memory profiling** finds allocation patterns and leaks
-   **HTTP profiling** enables live production profiling
-   **Goroutine profiling** shows running goroutines and their states
-   **Goroutine leak profiling** (Go 1.26+) automatically detects unreachable blocked goroutines
-   **Block/mutex profiling** diagnoses concurrency contention issues
-   **Execution tracing** reveals execution timelines and causality
-   **Flight recording** (Go 1.25+) provides lightweight continuous tracing for production
-   **`go tool pprof`** provides powerful analysis and visualization

Profiling is essential for writing high-performance Go code. By measuring before optimizing, focusing on real bottlenecks, and using the right tools for each problem, you can dramatically improve your application's performance. Remember: profile, optimize, measure, repeat.