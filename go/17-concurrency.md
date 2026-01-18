# Go > Concurrency

Go, with its strong support for concurrency, was designed from the ground up to take advantage of multi-core architectures to efficiently handle parallel processing. Before diving into goroutines and channels, let's first understand the difference between concurrency and parallelism.

**Concurrency** is about managing multiple tasks and making progress on them concurrently. It doesn't necessarily mean executing them simultaneously. It deals with the structure of the program and how tasks are organized. In a concurrent system, tasks can start, run, and complete in overlapping time periods, but they might not be actively executing at the same time.

**Parallelism**, on the other hand, involves actually executing multiple tasks simultaneously. It's about doing multiple things at the same time by leveraging multiple processors or cores. Parallelism is a way to achieve concurrency by physically running different tasks simultaneously.

With these definitions in mind, if you have written a Go program that can handle different tasks concurrently but your code runs on a single-core machine, no parallelism would happen; tasks would still run concurrently through time-slicing, but not in parallel.

Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time and data races cannot occur, by design.

## Understanding Processes and Threads

Before we dive into concurrency in Go, we need to understand the building blocks of program execution: processes and threads.

### What Is a Process?

When you run a program on your computer, the operating system creates a **process** for it. A process is an instance of a running program that has its own isolated memory space. Think of it like a container that holds everything your program needs to run: the code, variables, open files, and more. Each process is independent and isolated from other processes. For example, if you open two browser windows, you typically have two separate processes running. If one crashes, the other keeps running because they don't share memory.

### What Is a Thread?

A thread is a sequence of instructions stored in memory that gets executed line by line. Think of it as a to-do list that tells the computer what to do step by step. When you run any program, the operating system automatically creates a thred for it. This is called the **main thread**, and it runs your program from start to finish.

If you want your program to do multiple things at the same time (like downloading a file while updating the screen), you need more than 1 thread. To get additional threads, **you must write code to create them**. The operating system doesn't create extra threads automatically; that's your job as the programmer. So the division is simple:

-   **First thread**: Created automatically by the OS when your program starts
-   **Additional threads**: Created by you when you write code to create them

**What happens after threads are created:**

Once threads exist (whether the 1 automatic thread or multiple threads you created), the operating system takes over and manages them. The OS decides which thread runs on which CPU core and switches between threads very quickly. Your computer might have 4 CPU cores but your program could have 100 threads. The operating system rapidly switches which threads are running on those 4 cores, giving each thread a turn. This switching happens so fast it feels like all 100 threads are running simultaneously.

### OS Threads vs. Threads

In traditional programming languages (Java, C++, Python, C#), when you create a "thread" in your code, you're creating an **OS thread** an the operating system directly manages these threads, deciding when each one runs and on which CPU core. Characteristics of OS threads:

-   Created and managed by your operating system (Windows, macOS, Linux)
-   Each OS thread requires significant memory, typically 1-2 MB just for its stack (where function calls and local variables are stored)
-   Creating and switching between OS threads is relatively expensive because it involves the OS kernel
-   You can't create thousands of OS threads without running into memory and performance problems

Go introduces a different model. When you create a goroutine, you're **not** creating an OS thread. Instead:

-   Go creates a small pool of actual OS threads behind the scenes (typically one per CPU core)
-   Goroutines are "user-space threads" managed by Go's runtime scheduler, not the operating system
-   Go's scheduler multiplexes many goroutines onto a few OS threads
-   Each goroutine starts with only 2 KB of stack space (500x smaller than an OS thread)

This is why we call goroutines "lightweight threads". They're not managed by the OS, they use far less memory, and you can create millions of them without problems. In short:

-   **Traditional languages**: 1 thread in your code = 1 OS thread (1-to-1 mapping)
-   **Go**: Many goroutines are multiplexed onto few OS threads (M-to-N mapping meaning many goroutines called M are multiplexed onto fewer OS threads called N)

Think of it like transportation: OS threads are like taxis, where each person gets their own taxi which is expensive and limited by how many you can afford whereas goroutines are like a bus system, with many passengers (goroutines) sharing a few buses (OS threads), efficiently managed by a dispatcher, which in this case is Go's scheduler.

### What Do Threads Actually Execute?

If you write code in Go, what language does the thread execute? The answer is that threads don't execute any programming language. They execute machine code. Here's what happens when you run a Go program:

1. **You write Go code**: Your program is written in the Go programming language with functions, variables, and some logic
2. **Go compiler translates it**: The Go compiler converts your Go code into **machine code** (binary instructions like 0s and 1s that the CPU understands directly)
3. **OS creates a thread**: When you run the program, the operating system creates a thread, which is just a data structure containing a pointer to where the machine code starts and some memory for the program to use. Think of a thread as just a worker inside your computer. It's not a program or a language; it's more like a little machine that can follow instructions (it remembers where it left off in your program, which instruction to run next, carries function call history and temporary variables etc)
4. **Thread executes machine code**: The thread doesn't "run Go code". The thread is the container that the OS uses to execute that machine code

A thread is not software that "knows" how to execute instructions; it's simply a data structure that the OS maintains, holding the program counter (where it is in the code), the stack (temporary memory for function calls), CPU register values, and other bookkeeping information. When we say a thread "runs code", it doesn't actually interpret or understand the instructions; the CPU does the actual work by reading the machine code and performing operations like adding numbers or jumping to memory addresses. The thread merely provides the context and pointer so the CPU knows which instructions to execute and where to find the necessary data, much like a DVD tray holds a disc while the DVD player reads and displays the movie.

## What Is A Goroutine?

A goroutine is a lightweight thread managed by the Go runtime. Goroutines allow your program to run functions concurrently—think of them as different tasks running simultaneously within a program. This lets you handle multiple operations without waiting for each one to finish before starting the next. Unlike OS threads which typically require 1-2 MB of stack space, goroutines start with only 2 KB and can grow as needed. This means you can easily run thousands or even millions of goroutines in a single program, making Go exceptionally efficient for concurrent workloads.

## An Introduction to `WaitGroup`

`WaitGroup` from the `sync` package gives us the ability to create goroutines and wait for them to complete:

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

var wg = sync.WaitGroup{}

func main() {
	wg.Add(2)

	start := time.Now()

	go processTask("Task 1", 2)
	go processTask("Task 2", 1)

	fmt.Println("Active goroutines:", runtime.NumGoroutine())

	wg.Wait()

	elapsed := time.Since(start)
	fmt.Printf("Total time: %v\n", elapsed)
	fmt.Println("Active goroutines:", runtime.NumGoroutine())
}

func processTask(name string, seconds int) {
	defer wg.Done()

	time.Sleep(time.Duration(seconds) * time.Second)
	fmt.Printf("%s completed\n", name)
}
```

When you run this program, you'll see output like:

```text
Active goroutines: 3
Task 2 completed
Task 1 completed
Total time: 2.001272667s
Active goroutines: 1
```

`wg` is defined at the package level simply because we need to have access to it inside different functions. The way it works is that at the end of the `main` function we call `wg.Wait()` meaning the program must wait until all goroutines are done.

To turn function calls into goroutines, we prepend them with the `go` keyword. Since we have two goroutines that we need to wait for, at the very beginning of our `main` function we call `wg.Add(2)`, which tells Go to wait for 2 goroutines to complete.

As a last step, inside `processTask` we need to call `wg.Done()` to signal to the `WaitGroup` that the current function is done.

It's best practice to always call `wg.Add()` before launching the goroutine to avoid race conditions. Also, consider using `defer wg.Done()` as the first line in your goroutine to ensure it's always called; basically, it's safer and more idiomatic, especially in real-world code where the function might return early or encounter an error.

Let's analyze the program:

1. We create 2 goroutines by using the `go` keyword before calling `processTask`
2. The first `runtime.NumGoroutine()` prints 3 because we have the main goroutine plus the 2 we created
3. Tasks run concurrently; Task 1 takes 2 seconds and Task 2 takes 1 second, but they run at the same time
4. Total time is ~2 seconds, not 3 seconds (which would be the case if they ran sequentially)
5. As the two Go routines are completed, the second `runtime.NumGoroutine()` prints 1 because we only have the main goroutine

This demonstrates the key benefit of goroutines: when you have multiple independent tasks, they can run concurrently instead of waiting for each other. Task 2 finishes first (after 1 second) because it's shorter, even though we started Task 1 first. Without goroutines, if we called these functions normally, the total time would be 3 seconds because each task would wait for the previous one to finish. Let's validate that in action:

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {

	start := time.Now()

	processTask("Task 1", 2)
	processTask("Task 2", 1)

	fmt.Println("Active goroutines:", runtime.NumGoroutine())

	elapsed := time.Since(start)
	fmt.Printf("Total time: %v\n", elapsed)
	fmt.Println("Active goroutines:", runtime.NumGoroutine())
}

func processTask(name string, seconds int) {
	time.Sleep(time.Duration(seconds) * time.Second)
	fmt.Printf("%s completed\n", name)
}
```

When you run this program, you'll see output like:

```text
Task 1 completed
Task 2 completed
Active goroutines: 1
Total time: 3.001641834s
Active goroutines: 1
```

## I/O-bound versus CPU-bound Processes

### CPU-bound Processes

CPU-bound processes are limited by the CPU's computational capacity. Examples include encryption, decryption, or complex calculations that fully utilize the processor. Since CPU-bound processes use the CPU heavily, they are good candidates for parallelism or multi-processing, allowing multiple CPU cores to work simultaneously and speed up execution. In the following example, we are looping through a large number to mimic a CPU-intensive task:

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	start := time.Now()

	runtime.GOMAXPROCS(1)

	processTask()
	processTask()
	processTask()
	processTask()

	fmt.Printf("Total time: %v\n", time.Since(start)) // Total time: 2.06036775s
}

func processTask() {
	for i := 1; i < 1_000_000_000; i++ {
	}
}
```

`runtime.GOMAXPROCS(1)` sets how many CPU cores Go is allowed to use at the same time which in this case is 1. Think of it as:

```text
GOMAXPROCS = number of logical processors Go runtime can schedule goroutines on
```

Now let’s set `runtime.GOMAXPROCS(8)`, which allows Go to use eight logical processors. Surprisingly, the program still takes roughly the same amount of time to run. That's because the code executes sequentially, and increasing `GOMAXPROCS` does not improve performance when there is no parallel work. Now let's refactor the above program to make it concurrent using `WaitGroup`:

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

var wg = sync.WaitGroup{}

func main() {
	wg.Add(4)
	start := time.Now()

	runtime.GOMAXPROCS(1)

	go processTask()
	go processTask()
	go processTask()
	go processTask()

	wg.Wait()

	fmt.Printf("Total time: %v\n", time.Since(start))
}

func processTask() {
	defer wg.Done()
	for i := 1; i < 1_000_000_000; i++ {
	}
}
```

As shown above, we have four goroutines, but the number of processors is set to 1. In this case, we do not see any performance improvement because, although the code is concurrent, only one processor is available to execute the goroutines. When we increase the value using runtime.`GOMAXPROCS(4)`, Go can schedule the goroutines across multiple processors (cores), allowing them to run in parallel and improving performance (You can set `runtime.GOMAXPROCS(100)` even though your machine has only 8 cores because Go does not prevent you from asking for more processors than your CPU actually has. The operating system will simply time-slice the work across the available cores.)

In this program, setting `GOMAXPROCS(8)` does not provide any additional performance benefit because there are only four CPU-bound goroutines. At most, four goroutines can run in parallel, so once `GOMAXPROCS` is equal to or greater than four, all goroutines are already fully utilizing the available processors. Increasing `GOMAXPROCS` beyond that point does not improve performance, since there is no additional parallel work to schedule.

If you want Go to fully utilize all available CPU cores, set `runtime.GOMAXPROCS(runtime.NumCPU())`.

## I/O-bound Processes

I/O-bound processes are limited by I/O operations, such as reading from disk, making HTTP requests, or handling network traffic. The latency caused by these operations slows them down. Because they spend a lot of time waiting, I/O-bound processes are excellent candidates for concurrency, allowing other tasks to run while they wait. The following program sends multiple HTTP requests:

```go
package main

import (
	"fmt"
	"net/http"
	"runtime"
	"time"
)

func main() {
	start := time.Now()

	runtime.GOMAXPROCS(1)

	links := []string{
		"https://www.google.com",
		"https://www.amazon.com",
		"https://www.youtube.com",
		"https://www.wikipedia.org",
	}

	for _, link := range links {
		processTask(link)
	}

	fmt.Printf("Total time: %v\n", time.Since(start)) // Total time: 649.495708ms
}

func processTask(link string) {
	_, err := http.Get(link)
	if err != nil {
		return
	}

	fmt.Println(link)
}
```

Now if we set `runtime.GOMAXPROCS(4)` we won't get any performance improvement because the code itself is procedural. . Now let's refactor the above code to make it concurrent:

```go
package main

import (
	"fmt"
	"net/http"
	"runtime"
	"sync"
	"time"
)

var wg = sync.WaitGroup{}

func main() {
	wg.Add(4)
	start := time.Now()

	runtime.GOMAXPROCS(1)

	links := []string{
		"https://www.google.com",
		"https://www.amazon.com",
		"https://www.youtube.com",
		"https://www.wikipedia.org",
	}

	for _, link := range links {
		go processTask(link)
	}

	wg.Wait()

	fmt.Printf("Total time: %v\n", time.Since(start)) // Total time: 168.344833ms
}

func processTask(link string) {
	defer wg.Done()
	_, err := http.Get(link)
	if err != nil {
		return
	}

	fmt.Println(link)
}
```

As shown above, even with only one processor, I/O-bound operations can achieve significant performance improvements. For I/O-bound processes, the bottleneck is not the CPU but the latency of external operations, such as network requests or disk reads. While a goroutine is waiting for an HTTP response or a file read to complete, the CPU is idle and can switch to executing another goroutine. This means that even with a single processor, multiple I/O-bound tasks can run concurrently because they spend most of their time waiting rather than using the CPU. The Go scheduler efficiently switches between these waiting goroutines, allowing the program to make progress on other tasks without requiring multiple CPU cores. As a result, concurrency improves performance for I/O-bound workloads, even when `GOMAXPROCS` is set to 1.

## Channels

Channels are the pipes that connect concurrent goroutines. You can send values into channels from one goroutine and receive those values in another goroutine.

**Key concept**: Unbuffered channels are blocking—they require both a sender and receiver to be ready simultaneously. In the following program, we are creating a channel called `c` which can hold any value of type `int`, then in the next line we are adding `78` to our channel and this is the line where our channel blocks.

```go
package main

import "fmt"

func main() {
	c := make(chan int)

	c <- 78

	fmt.Println(<-c)
}
```

If we try to run the above code we will get:

```text
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:8 +0x38
exit status 2
```

This error occurs because you're trying to send a value into a channel but there's no goroutine (concurrent function) ready to receive it.

**Why this happens**: When you create a channel with `c := make(chan int)`, it's an **unbuffered channel**, meaning it requires both a sender and a receiver to be ready simultaneously for communication. This is a synchronization point—the sender blocks until a receiver is ready, and the receiver blocks until a sender is ready.

When you execute `c <- 78`, the main goroutine tries to send 78 into the channel, but since there's no other goroutine ready to receive from it, the program deadlocks. All goroutines are blocked waiting for something that will never happen.

To fix this issue, you have two options:

**Option 1: Use Goroutines** - Create a separate goroutine to send the value into the channel while the main goroutine is waiting to receive it.

```go
package main

import "fmt"

func main() {
	c := make(chan int)

	go func() {
		c <- 78
	}()

	fmt.Println(<-c)
}
```

**Option 2: Buffered Channel** - Create a buffered channel that can hold one value without requiring the sender and receiver to be ready simultaneously.

```go
package main

import "fmt"

func main() {
	c := make(chan int, 1) // Buffer size of 1

	c <- 78 // Doesn't block because buffer has space

	fmt.Println(<-c) // Receives from buffer
}
```

Both of these solutions prevent the deadlock. The first uses concurrency (goroutines), while the second uses buffering to decouple sending and receiving.

### Channel Directionality

Channels can be made to specifically send values, receive values, or both. This adds type safety and makes your intent clearer:

```go
package main

import "fmt"

func main() {
	c := make(chan int) // send & receive
	c1 := make(chan<- int) // send
	c2 := make(<-chan int) // receive

	fmt.Printf("%T\n", c)
	fmt.Printf("%T\n", c1)
	fmt.Printf("%T\n", c2)
}
```

This outputs:

```text
chan int       // bidirectional
chan<- int     // send-only
<-chan int     // receive-only
```

A more practical example of send-only and receive-only channels is as follows:

```go
package main

import "fmt"

func sender(c chan<- int) {
	c <- 7
}

func receiver(c <-chan int) {
	fmt.Println(<-c)
}

func main() {
	c := make(chan int)

	sender(c)
	receiver(c)
}
```

Since channels are reference types, the same channel `c` is passed to both functions. If we run the above program, it outputs:

```text
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.sender(...)
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:6
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:16 +0x3c
exit status 2
```

The reason is a synchronization problem between the sender and receiver. When the `sender` function attempts to send a value to the channel `c <- 7`, it blocks because there's no other goroutine ready to receive the value from the channel at that moment. The program never gets to the `receiver` call because it's stuck on the `sender` call.

To fix this, you need to run these functions concurrently using goroutines. Use the `go` keyword to launch these functions as goroutines:

```go
package main

import "fmt"

func sender(c chan<- int) {
	c <- 7
}

func receiver(c <-chan int) {
	fmt.Println(<-c)
}

func main() {
	c := make(chan int)

	go sender(c)
	go receiver(c)
}
```

This fixed the deadlock, but now we have another issue—nothing is outputted in the terminal. The reason you don't see any output is that the `main` function finishes execution before the goroutines `sender` and `receiver` get a chance to execute. In Go, the `main` function **doesn't wait** for goroutines to finish by default.

To fix this, we have two options:

1. **Use `WaitGroup`** to wait for both goroutines to complete
2. **Remove the `go` keyword from `receiver`** - The `receiver` function will then run in the `main` goroutine and block until a value is available on the channel. If a value is already sent by the `sender` goroutine, it proceeds; otherwise, it blocks until a value is sent.

The second option is simpler for this example:

```go
func main() {
	c := make(chan int)

	go sender(c)  // Runs concurrently
	receiver(c)   // Blocks main until it receives a value
}
```

## How to Use Channels with Range

We can loop through a channel's values using a `range` loop. This is useful when you want to receive multiple values from a channel:

```go
package main

import (
	"fmt"
)

func main() {
	c := make(chan int)

	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
	}()

	for v := range c {
		fmt.Println(v)
	}

	fmt.Println("program exited successfully")
}
```

The `for range` loop iterates over all channel values until the channel is closed. Let's see what we get in the output:

```text
0
1
2
3
4
5
6
7
8
9
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:16 +0xc0
exit status 2
```

In the above code, the `range` loop iterates over the channel until it's closed. The problem here is that we have not explicitly closed our channel, that's why `range` keeps waiting for more values even though there are no more values to send, and we get a deadlock error. To fix this, we need to close our channel when we are done sending values:

```go
go func() {
	for i := 0; i < 10; i++ {
		c <- i
	}

	close(c)
}()
```

Now it outputs:

```text
0
1
2
3
4
5
6
7
8
9
program existed successfully
```

## How to Use Channels with `select` Statement

The `select` statement in Go is similar to a `switch` statement, but it's designed specifically for communication between goroutines via channels. While `switch` works with multiple expressions to determine the flow of control, `select` deals with communication operations on channels.

**Key characteristics**:

-   A `select` statement allows you to wait on multiple channel operations simultaneously
-   If none of the channels are ready, the `select` statement blocks until at least one of the channels is ready to proceed
-   If multiple channels are ready, Go randomly selects one (this prevents starvation)

```go
package main

import (
	"fmt"
	"time"
)

func create(even, odd chan<- int, quit chan<- bool) {
	for i := 0; i < 10; i++ {
		if i%2 == 0 {
			time.Sleep(time.Second * 1)
			even <- i
		} else {
			time.Sleep(time.Second * 3)
			odd <- i
		}
	}

	quit <- true
}

func show(even, odd <-chan int, quit <-chan bool) {
	for {
		select {
		case e := <-even:
			fmt.Println("Even:", e)
		case o := <-odd:
			fmt.Println("Odd:", o)
		case q := <-quit:
			fmt.Println("Done", q)

			return
		}
	}
}

func main() {
	even := make(chan int)
	odd := make(chan int)
	quit := make(chan bool)

	go create(even, odd, quit)
	show(even, odd, quit)
}
```

In the above program, the `select` statement listens to three channels: `even`, `odd`, and `quit`. When any of these channels are ready to be read from, the corresponding case inside the `select` will execute:

-   If a value is received from the `even` channel, it prints it as an even number
-   If a value is received from the `odd` channel, it prints it as an odd number
-   If a value is received from the `quit` channel, it prints "Done" and exits the `show` function

**Important**: The `select` statement doesn't operate on values like a `switch` statement does. Instead, it chooses which channel operation to proceed with based on the readiness of the channels involved. It's a powerful construct for handling concurrent communication between goroutines.

## Another Example of Channels in Go

Since functions are called one after the other in sequence, if we have a long-running task, it will block the execution of other functions that come next:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	greet("Nice to meet you")
	greet("How are you?")
	slowGreet("How ... are ... you ... ?")
	greet("The final greeting")
}

func greet(phrase string) {
	fmt.Println("Hi", phrase)
}

func slowGreet(phrase string) {
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)
}
```

We won't be able to see the output of the last `greet()` invocation right away because the previous function (`slowGreet()`) takes 3 seconds to complete.

To tackle this issue and run all functions concurrently, we need to use the `go` keyword as follows:

```go
func main() {
	go greet("Nice to meet you")
	go greet("How are you?")
	go slowGreet("How ... are ... you ... ?")
	go greet("The final greeting")
}
```

But if we run the program now, we won't see any output in the terminal and the program exits right away.

**Why this happens**: By adding the `go` keyword, you tell Go that you want to run those functions as goroutines, which means they will run concurrently instead of one after the other. The idea behind running a function as a goroutine is that it will run in a non-blocking way, so the next function can immediately be invoked.

The reason the program exits right away is that the `main()` function does not wait for goroutines to complete. The `main()` function dispatches the goroutines and immediately continues to the end of the function, at which point the program exits.

The solution to this problem is to use channels or `WaitGroup` to synchronize. Simply put, a channel is a value that can be used as a communication mechanism when working with goroutines:

```go
func main() {
	done := make(chan bool)
	go slowGreet("How ... are ... you ... ?", done)
	<-done
}

func slowGreet(phrase string, doneChannel chan bool) {
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)

	doneChannel <- true // The <- points to the direction that the flag should flow
}
```

This channel lets Go know whether the function process is done or not.

When the `slowGreet()` function completes, it sends a flag through its channel to the place where we started the goroutine, which is the following line:

```go
go slowGreet("How ... are ... you ... ?")
```

Go will exit only after data comes out of the channel on this line `<-done`. This is a **blocking receive**—the program waits at this line until the channel receives a value.

We can also place the received value inside `fmt.Println(<-done)` or simply use `<-done` to wait without printing.

We can use a single channel for multiple goroutines. To do that, we must ensure the channel is created before launching any goroutine and that all goroutines accept the channel as a parameter:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	done := make(chan bool)

	go greet("First call", done)
	go greet("Second call", done)
	go slowGreet("Delayed call", done)
	go greet("Last call", done)

	<-done
}

func slowGreet(phrase string, doneChannel chan bool) {
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)

	doneChannel <- true
}

func greet(phrase string, doneChannel chan bool) {
	fmt.Println("Hi", phrase)

	doneChannel <- true
}

```

Keep in mind that we can use the same channel for multiple goroutines because channels are designed to be communication devices that can receive multiple values.

However, if we run the above program, we will get unpredictable results:

```text
Hi Last call
```

Or in another execution we get:

```text
Hi Last call
Hi First call
Hi Second call
```

The reason we're getting this unpredictable result is that **we're only waiting for one channel value** even though we launched four goroutines. The program exits as soon as the first goroutine sends its value to the channel. To tackle this issue, we need to wait for all four goroutines:

```go
func main() {
	done := make(chan bool)

	go greet("First call", done)
	go greet("Second call", done)
	go slowGreet("Delayed call", done)
	go greet("Last call", done)

	<-done
	<-done
	<-done
	<-done
}
```

In the output we have:

```text
Hi Last call
Hi Second call
Hi First call
Hi Delayed call
```

**Note on output order**: The order of the output doesn't match the order of function calls. This is because all those goroutines execute concurrently, and the output reflects the completion order. The `greet()` function call with `Last call` might finish way sooner than the one with `First call` because goroutines are scheduled independently.

**Rule of thumb**: If you use the same channel for multiple goroutines, you must wait for as many values as you have goroutines.

**Problem with this approach**: This way of handling goroutines is not scalable and is error-prone because you might forget to match the number of receives with the number of goroutines. There are better alternatives:

```go
func main() {
	dones := make([]chan bool, 4)

	dones[0] = make(chan bool)
	go greet("First call", dones[0])

	dones[1] = make(chan bool)
	go greet("Second call", dones[1])

	dones[2] = make(chan bool)
	go slowGreet("Delayed call", dones[2])

	dones[3] = make(chan bool)
	go greet("Last call", dones[3])

	for _, done := range dones {
		<-done
	}
}
```

The above approach is better than manually writing multiple `<-done` statements, but it's still not ideal. **The recommended approach is to use `WaitGroup`** (shown earlier in this tutorial) or to use a `for range` loop over the channel:

```go
func main() {
	done := make(chan bool)
	go greet("First call", done)
	go greet("Second call", done)
	go slowGreet("Delayed call", done)
	go greet("Last call", done)

	for isDone := range done {
		fmt.Println(isDone)
	}
}
```

As shown above, we can loop through the channel, but in the output we will get an error:

```text
Hi First call
Hi Last call
true
true
Hi Second call
true
Hi Delayed call
true
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/Users/behzadmoradi/Documents/projects/go-tmp/main.go:15 +0x184
exit status 2
```

We get this error because Go does not know when the channel is out of values. The `for range` loop keeps waiting for new values to be sent, but eventually there are no more values, resulting in a deadlock.

**Antipattern - Don't do this**: Closing the channel in the function that finishes last:

```go
func slowGreet(phrase string, doneChannel chan bool) {
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)

	doneChannel <- true
	close(doneChannel) // BAD: Assumes this finishes last
}
```

**Why this is bad**: This only works if you know for certain which operation takes longest, and it's fragile—any change to timing breaks the code.

**Better approach**: Use `WaitGroup` instead:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func main() {
	wg.Add(4)

	go greet("First call")
	go greet("Second call")
	go slowGreet("Delayed call")
	go greet("Last call")

	wg.Wait() // Clean, clear, correct
}

func slowGreet(phrase string) {
	defer wg.Done()
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)
}

func greet(phrase string) {
	defer wg.Done()
	fmt.Println("Hi", phrase)
}
```

This is the idiomatic Go way to wait for multiple goroutines to complete.

## Race Condition & Mutex

A mutex, short for "mutual exclusion," is a synchronization mechanism used in concurrent programming to ensure that only one thread or goroutine can access a shared resource or critical section at a time. The purpose of a mutex is to prevent data races and maintain the consistency of shared data. This is how mutex works:

**Locking**: When a goroutine wants to access a critical section or modify a shared resource, it must acquire the mutex by calling the `Lock` method. If the mutex is currently held by another goroutine, the requesting goroutine will be blocked until the mutex is released by the current holder.

**Critical Section**: Once a goroutine has acquired the mutex, it can safely access the shared resource or execute a critical section of code. The mutex ensures that only one goroutine is in this critical section at any given time.

**Unlocking**: After completing the critical section, the goroutine releases the mutex by calling the `Unlock` method. This allows other goroutines waiting for the mutex to acquire it and proceed.

### Understanding Race Conditions

In the context of Go, a race condition occurs when two or more goroutines access shared data concurrently, and at least one of them modifies the data. The behavior of the program becomes unpredictable because the outcome depends on the timing and order of execution of the goroutines. Race conditions can lead to unexpected and erroneous behavior in a program. The following program shows race condition in action:

```go
package main

import (
	"fmt"
	"sync"
)

var counter int32 = 0
var wg = sync.WaitGroup{}

func main() {
	wg.Add(2)

	go processTask()
	go processTask()

	wg.Wait()

	fmt.Printf("Counter value: %v\n", counter)
}

func processTask() {
	defer wg.Done()

	for range 1_000_000 {
		counter++
	}
}
```

Each time you run the above program, you would get a different result in the terminal. Go's runtime includes a race detector that helps identify such issues during development. The race detector can be enabled by using the `-race` flag when compiling or running a Go program:

```sh
go run -race .
```

Let's first see part of the output, then analyze it:

```text
==================
WARNING: DATA RACE
Read at 0x000100f70a38 by goroutine 8:
  main.processTask()
      main.go:29 +0x84

Previous write at 0x000100f70a38 by goroutine 9:
  main.processTask()
      main.go:29 +0x9c

Goroutine 8 (running) created at:
  main.main()
      main.go:16 +0x4c

Goroutine 9 (running) created at:
  main.main()
      main.go:17 +0x58
==================
==================
WARNING: DATA RACE
Write at 0x000100f70a38 by goroutine 8:
  main.processTask()
      main.go:29 +0x9c

Previous write at 0x000100f70a38 by goroutine 9:
  main.processTask()
      main.go:29 +0x9c

Goroutine 8 (running) created at:
  main.main()
      main.go:16 +0x4c

Goroutine 9 (running) created at:
  main.main()
      main.go:17 +0x58
==================
```

As shown above, both GoRoutine 8 and 9 are trying to read from and write to the same memory location (`0x000100f70a38`). Also the above warning shows that you goroutine 8 was started by line 16 and goroutine 9 was started by line 17. We could say that the first warning is that one goroutine read `counter` while another was writing; the second warning shows that both goroutines were writing at the same time.

To address this issue and avoid race conditions, you should use synchronization mechanisms like a **Mutex** (which stands for **Mut**ually **ex**clusive). Here is how we can fix it using `Mutex`:

```go
package main

import (
	"fmt"
	"sync"
)

var counter int32 = 0
var mutex = sync.Mutex{}
var wg = sync.WaitGroup{}

func main() {
	wg.Add(2)

	go processTask()
	go processTask()

	wg.Wait()

	fmt.Printf("Counter value: %v\n", counter)
}

func processTask() {
	defer wg.Done()

	for range 1_000_000 {
		mutex.Lock()
		counter++
		mutex.Unlock()
	}
}
```

In this modified version, `mutex` is used to ensure that only one goroutine can modify the `counter` variable at a time. The `Lock` and `Unlock` methods of the `mutex` guarantee the atomicity of the critical section, preventing race conditions and ensuring that the final value of counter is as expected. Now if we run `go run -race .`, we'll get `Counter value: 2000000`.

As an experiment, let's place the `mutex` variable inside the `processTask` function:

```go
func processTask() {
	var mutex = sync.Mutex{}

	defer wg.Done()

	for range 1_000_000 {
		mutex.Lock()
		counter++
		mutex.Unlock()
	}
}
```

If we run the program, the race condition scenario is back and the reason is that by calling the `processTask` function two times, we are creating two separate `mutex` variables; one for each function call and the locking and unlocking functionalities have no effect on each other. Each function call has its own mutex instance, so they don't actually coordinate access to the shared `counter` variable. This demonstrates why mutexes must be shared between goroutines to be effective.

## What Is Atomicity?

In concurrent programming, atomicity means: "Go it all at once, or not at all." An atomic operation is an operation that:

-   Cannot be split into smaller steps
-   Cannot be interrupted by another goroutine or thread
-   Appears to happen instantly to everything else in the program
-   Other goroutines never see the operation “half done”.

We have seen so far that when multiple goroutines work with the same variable at the same time, problems can happen Which is called a race condition. When an operation is atomic, all its steps happen as one unbreakable action. This guarantees that:

-   Only one goroutine can modify the value at a time
-   No updates are lost
-   The final result is correct

In the context of shared variables, atomicity is crucial to prevent race conditions. A race condition occurs when multiple threads or goroutines access shared data concurrently, and at least one of them modifies the data. If the operations on the shared data are not atomic, the interleaving of operations from different threads can lead to unexpected and incorrect results.

So far, we have seen how we can use the `mutex` synchronization mechanism to achieve atomic operations. `sync/atomic` package is another option we can use to achieve atomic operations. Basically, it provides atomic operations for basic types like integers, ensuring that operations like increments, decrements, swaps, etc., are atomic and free from race conditions. We can rewrite the above program using the `sync/atomic` package as follows:

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var counter int32 = 0
var wg = sync.WaitGroup{}

func main() {
	wg.Add(2)

	go processTask()
	go processTask()

	wg.Wait()

	fmt.Printf("Counter value: %v\n", counter)
}

func processTask() {
	defer wg.Done()

	for range 1_000_000 {
		atomic.AddInt32(&counter, 1)
	}
}
```

Still there won't be any race condition and no matter how many times we run it, we'll get the same result: `Counter value: 2000000`.

## Condition Variables

So far, we’ve learned how to protect shared data using a mutex and how to make operations atomic using the `sync/atomic` package, which helps prevent race conditions when multiple goroutines access the same data. However, there is still an important problem we haven't solved yet: how can goroutines efficiently **wait for a specific condition** to become true without wasting CPU resources or constantly checking the same value in a loop?

A mutex can protect data, but it cannot coordinate timing. Consider this scenario that what if one goroutine needs to wait until another goroutine finishes some work or changes a value? A common (but bad) solution is busy waiting:

```go
for counter < 10 {
	// keep checking
}
```

The problem with the above struct is that it wastes CPU, it doesn't scale, and it's inefficient. Instead, we need a struct that kills the machine "Go to sleep until something changes then wake me up." That's exactly what Condition Variables are for. A condition variable allows goroutines to:

-   Wait until a condition becomes true
-   Sleep efficiently without using CPU
-   Wake up when another goroutine signals that something has changed

In Go, condition variables are implemented using `sync.Cond` which gives us the ability to use the following method:

-   `Wait()`: Puts the goroutine to sleep (releases the mutex while waiting)
-   `Signal()`: Wakes one waiting goroutine
-   `Broadcast()`: Wakes all waiting goroutines

Now we are going to extend our counter example in a way that one goroutine increments the `counter` variable, another one waits until `counter` is equal to `2_000_000`. Once the condition is met, the awaiting goroutine continues.

With this intro, let's see how we can extend our program to use condition variables:

```go
package main

import (
	"fmt"
	"sync"
)

var counter int32 = 0
var mutex = sync.Mutex{}
var wg = sync.WaitGroup{}
var cond = sync.NewCond(&mutex)

func main() {
	wg.Add(3)

	go processTask()
	go processTask()

	go waitForCounter()

	wg.Wait()
}

func processTask() {
	defer wg.Done()

	for range 1_000_000 {
		mutex.Lock()
		counter++

		if counter == 2_000_000 {
			cond.Signal()
		}

		mutex.Unlock()
	}
}

func waitForCounter() {
	defer wg.Done()
	mutex.Lock()
	for counter < 2_000_000 {
		cond.Wait()
	}

	fmt.Println("Counter reached:", counter)
	mutex.Unlock()
}
```

Each goroutine runs this loop `1_000_000` times and there are two goroutines so `1_000_000 + 1_000_000 = 2_000_000`. Here is what happens under the hood:

-   The `waitForCounter` goroutine locks the `mutex` so it can safely read counter
-   It checks the value of `counter`. If the condition is not met, it calls `Wait()`
-   Calling `Wait()` releases the `mutex` so other goroutines can continue
-   The `waitForCounter` goroutine is then put to **sleep** and uses no CPU
-   Another goroutine locks the `mutex` and updates `counter`
-   Once the update may satisfy the condition, it calls `Signal()`
-   The sleeping `waitForCounter` goroutine wakes up and gefore continuing, it re-acquires the `mutex`
-   It checks the condition again (this is very important). If the condition is now true, execution continues; otherwise, it goes back to waiting by calling `Wait()`

### `Signal` versus `Broadcast`

`Signal` wakes only one waiting goroutine, while `Broadcast` awakes all waiting goroutines. Therefore, to observe the effect of Broadcast, there must be more than one goroutine waiting on the condition; otherwise, it behaves the same as `Signal`:

```go
package main

import (
	"fmt"
	"sync"
)

var counter int32 = 0
var mutex = sync.Mutex{}
var wg = sync.WaitGroup{}
var cond = sync.NewCond(&mutex)

func main() {
	wg.Add(4)

	go processTask()
	go processTask()

	go waitForCounter()
	go waitForCounter2()

	wg.Wait()
}

func processTask() {
	defer wg.Done()

	for range 1_000_000 {
		mutex.Lock()
		counter++

		if counter == 2_000_000 {
			cond.Broadcast()
		}

		mutex.Unlock()
	}
}

func waitForCounter() {
	defer wg.Done()
	mutex.Lock()
	for counter < 2_000_000 {
		cond.Wait()
	}

	fmt.Println("Counter reached:", counter)
	mutex.Unlock()
}

func waitForCounter2() {
	defer wg.Done()
	mutex.Lock()
	for counter < 2_000_000 {
		cond.Wait()
	}

	fmt.Println("Counter reached:", counter)
	mutex.Unlock()
}
```

We have introduced a second listener, `waitForCounter2`, which waits on the same condition variable as `waitForCounter`. Both goroutines block until the shared condition (`counter >= 2_000_000`) becomes true. Because multiple goroutines may be waiting on the same condition, `cond.Broadcast()` is used to wake all waiting goroutines once the condition is satisfied. Each awakened goroutine re-acquires the mutex, re-checks the condition, and then proceeds safely (If `cond.Broadcast()` is replaced with `cond.Signal()`, only one waiting goroutine is awakened. The other remains blocked indefinitely, causing the program to deadlock when `wg.Wait()` is called.)

## The `context` Package: Managing Goroutine Lifecycles

The `context` package is fundamental to modern Go concurrency. It provides a way to carry deadlines, cancellation signals, and request-scoped values across API boundaries and between goroutines. **Mastering context can eliminate 90% of potential goroutine leaks.**

### Why Context Matters

Without proper cancellation, goroutines can leak—they keep running even when their work is no longer needed, consuming memory and CPU. Context solves this by providing a standard way to signal cancellation.

### Basic Context Usage

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("%s: stopped due to cancellation\n", name)
			return
		default:
			fmt.Printf("%s: working...\n", name)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	// Create a context that can be cancelled
	ctx, cancel := context.WithCancel(context.Background())

	go doWork(ctx, "Worker 1")
	go doWork(ctx, "Worker 2")

	// Let them work for 2 seconds
	time.Sleep(2 * time.Second)

	// Cancel the context - stops all goroutines
	cancel()

	// Give goroutines time to stop
	time.Sleep(500 * time.Millisecond)
	fmt.Println("Main: done")
}
```

### Context with Timeout

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func fetchData(ctx context.Context) error {
	// Simulate a long-running operation
	select {
	case <-time.After(3 * time.Second):
		fmt.Println("Data fetched successfully")
		return nil
	case <-ctx.Done():
		return ctx.Err() // Returns context.DeadlineExceeded or context.Canceled
	}
}

func main() {
	// Create a context with 1-second timeout
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel() // Always call cancel to release resources

	err := fetchData(ctx)
	if err != nil {
		fmt.Printf("Error: %v\n", err) // Prints: context deadline exceeded
	}
}
```

### Context Best Practices

1. **Always call cancel**: Even if the context will expire, call `cancel()` to release resources. Use `defer cancel()` immediately after creating the context.

2. **Pass context as first parameter**: By convention, context is always the first parameter in function signatures:

    ```go
    func doSomething(ctx context.Context, data string) error
    ```

3. **Don't store context in structs**: Contexts are designed to be short-lived and passed explicitly through call chains.

4. **Use select with ctx.Done()**: Always provide a way to cancel long-running operations:

    ```go
    select {
    case <-ctx.Done():
        return ctx.Err()
    case result := <-workChan:
        // process result
    }
    ```

5. **Context values are for request-scoped data only**: Don't use context.Value for passing optional parameters. Use it only for request-scoped data like request IDs, authentication tokens, etc.

## Preventing Goroutine Leaks

Goroutine leaks occur when goroutines are started but never properly terminated, causing memory leaks and resource exhaustion in long-running applications. Here's how to prevent them:

### Common Leak Scenario 1: Blocked Channel Send

```go
// BAD: This goroutine leaks if no one receives
func leakyFunction() {
	ch := make(chan int)
	go func() {
		result := expensiveOperation()
		ch <- result // Blocks forever if main doesn't receive
	}()
	// If we return here without receiving, goroutine leaks
}
```

**Solution**: Use context for cancellation:

```go
// GOOD: Goroutine can be cancelled
func nonLeakyFunction(ctx context.Context) {
	ch := make(chan int)
	go func() {
		result := expensiveOperation()
		select {
		case ch <- result:
			// Sent successfully
		case <-ctx.Done():
			// Context cancelled, exit goroutine
			return
		}
	}()
}
```

### Common Leak Scenario 2: Infinite Loop Without Exit

```go
// BAD: No way to stop this goroutine
func startWorker() {
	go func() {
		for {
			doWork()
			time.Sleep(time.Second)
		}
	}()
}
```

**Solution**: Use context to signal shutdown:

```go
// GOOD: Goroutine can be gracefully stopped
func startWorker(ctx context.Context) {
	go func() {
		for {
			select {
			case <-ctx.Done():
				return // Exit goroutine
			default:
				doWork()
				time.Sleep(time.Second)
			}
		}
	}()
}
```

### Common Leak Scenario 3: Goroutine Waiting on Channel That Never Closes

```go
// BAD: If producer stops sending without closing channel, consumer leaks
func consumer(ch chan int) {
	go func() {
		for val := range ch { // Blocks forever if channel never closes
			process(val)
		}
	}()
}
```

**Solution**: Always close channels when done, or use context:

```go
// GOOD: Multiple exit strategies
func consumer(ctx context.Context, ch chan int) {
	go func() {
		for {
			select {
			case val, ok := <-ch:
				if !ok {
					return // Channel closed
				}
				process(val)
			case <-ctx.Done():
				return // Context cancelled
			}
		}
	}()
}
```

### Detecting Goroutine Leaks

Use `runtime.NumGoroutine()` to monitor goroutine count:

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	fmt.Printf("Initial goroutines: %d\n", runtime.NumGoroutine())

	for i := 0; i < 10; i++ {
		go func() {
			time.Sleep(10 * time.Second) // Long-running
		}()
	}

	fmt.Printf("After launching 10: %d\n", runtime.NumGoroutine())

	// In production, use monitoring to track this metric
}
```

**Production tip**: Monitor `runtime.NumGoroutine()` in your applications. A steadily increasing goroutine count indicates a leak.

## What Are The Differences between `sync.Mutex` and `sync/atomic`?

Both `sync.Mutex` and the `sync/atomic` package can be used to achieve synchronization and prevent race conditions in concurrent programming. However, there are differences between them, and the choice depends on the specific requirements of your program.

### `sync.Mutex`

**Use case**: Protecting larger sections of code (critical sections) where multiple operations need to be performed atomically.

**How it works**: Provides a lock and unlock mechanism. When a goroutine acquires a lock, other goroutines must wait.

**Overhead**: Higher overhead because it requires acquiring and releasing locks, and may involve context switching between goroutines.

**Example use**: Protecting access to a complex data structure like a map or performing multiple related operations that must happen atomically.

### `sync/atomic`

**Use case**: Fine-grained operations on individual variables (integers, pointers, etc.).

**How it works**: Provides atomic operations directly on memory (increments, swaps, compare-and-swap, etc.) without locks.

**Overhead**: Lower overhead because it directly operates on memory using CPU-level atomic instructions without acquiring locks.

**Example use**: Incrementing a counter, updating a flag, or swapping a value.

### Summary

-   **Use `sync.Mutex`** when you need to protect larger sections of code or perform complex synchronization
-   **Use `sync/atomic`** when you have specific atomic operations on individual variables and need lower-level, lightweight synchronization
-   For simple counters or flags, `sync/atomic` is faster and more efficient
-   For complex operations involving multiple variables, use `sync.Mutex`

## Advanced Concurrency Patterns

### Worker Pool Pattern

The worker pool pattern limits the number of concurrent workers, which is essential for controlling resource usage when processing many tasks:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job)
		time.Sleep(time.Second) // Simulate work
		results <- job * 2
	}
}

func main() {
	const numWorkers = 3
	const numJobs = 10

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	var wg sync.WaitGroup

	// Start workers
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go worker(i, jobs, results, &wg)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs) // Important: close jobs channel when done sending

	// Wait for workers to finish
	wg.Wait()
	close(results) // Close results after all workers are done

	// Collect results
	for result := range results {
		fmt.Printf("Result: %d\n", result)
	}
}
```

**Key points**:

-   Fixed number of workers prevents resource exhaustion
-   Jobs channel buffers work items
-   Closing the jobs channel signals workers to exit
-   WaitGroup ensures all workers finish before closing results

### Fan-Out, Fan-In Pattern

Fan-out distributes work to multiple goroutines; fan-in combines their results:

```go
package main

import (
	"fmt"
	"sync"
)

func generator(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			out <- n
		}
	}()
	return out
}

func square(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			out <- n * n
		}
	}()
	return out
}

// Fan-in: merge multiple channels into one
func merge(channels ...<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup

	wg.Add(len(channels))
	for _, ch := range channels {
		go func(c <-chan int) {
			defer wg.Done()
			for n := range c {
				out <- n
			}
		}(ch)
	}

	go func() {
		wg.Wait()
		close(out)
	}()

	return out
}

func main() {
	in := generator(1, 2, 3, 4, 5, 6, 7, 8)

	// Fan-out: distribute work to multiple goroutines
	c1 := square(in)
	c2 := square(in)
	c3 := square(in)

	// Fan-in: combine results
	for result := range merge(c1, c2, c3) {
		fmt.Println(result)
	}
}
```

### Pipeline Pattern with Cancellation

```go
package main

import (
	"context"
	"fmt"
)

func generator(ctx context.Context, nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			select {
			case out <- n:
			case <-ctx.Done():
				return // Cancelled
			}
		}
	}()
	return out
}

func square(ctx context.Context, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			select {
			case out <- n * n:
			case <-ctx.Done():
				return // Cancelled
			}
		}
	}()
	return out
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	// Build pipeline
	in := generator(ctx, nums...)
	out := square(ctx, in)

	// Read first 3 results, then cancel
	for i := 0; i < 3; i++ {
		fmt.Println(<-out)
	}

	cancel() // Cancel remaining work
	fmt.Println("Pipeline cancelled")
}
```

**Key insight**: Each stage checks `ctx.Done()` to enable graceful cancellation of the entire pipeline.

### Using `errgroup` for Error Handling

The `golang.org/x/sync/errgroup` package extends `WaitGroup` with error propagation:

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"time"

	"golang.org/x/sync/errgroup"
)

func task1(ctx context.Context) error {
	time.Sleep(1 * time.Second)
	fmt.Println("Task 1 completed")
	return nil
}

func task2(ctx context.Context) error {
	time.Sleep(2 * time.Second)
	return errors.New("task 2 failed")
}

func task3(ctx context.Context) error {
	time.Sleep(3 * time.Second)
	fmt.Println("Task 3 completed")
	return nil
}

func main() {
	g, ctx := errgroup.WithContext(context.Background())

	g.Go(func() error { return task1(ctx) })
	g.Go(func() error { return task2(ctx) })
	g.Go(func() error { return task3(ctx) })

	// Wait for all tasks; returns first non-nil error
	if err := g.Wait(); err != nil {
		fmt.Printf("Error: %v\n", err)
	}
}
```

**Benefits**:

-   Automatic context cancellation on first error
-   Cleaner error handling than manual WaitGroup
-   First-error-wins semantics

## Performance Considerations

### Buffered vs Unbuffered Channels

**Unbuffered channels** (`make(chan T)`):

-   Provide strict synchronization
-   Sender blocks until receiver is ready
-   Better when you need point-to-point coordination
-   Slightly better performance for synchronization

**Buffered channels** (`make(chan T, n)`):

-   Sender blocks only when buffer is full
-   Receiver blocks only when buffer is empty
-   Better for handling bursts of work
-   Can reduce context switches

```go
// Unbuffered - strict synchronization
ch := make(chan int)

// Buffered - allows n sends without receiver
ch := make(chan int, 100)
```

**Guideline**: Start with unbuffered channels for clarity. Add buffering only when profiling shows it helps.

### Goroutine Creation Cost

-   Goroutines are cheap: ~2 KB initial stack
-   Can run millions of goroutines
-   However, creating goroutines isn't free
-   Use worker pools for high-volume workloads

### Using the Race Detector

Always test concurrent code with the race detector:

```bash
go test -race
go run -race main.go
go build -race
```

The race detector finds data races at runtime. It's invaluable for concurrent code.

### Benchmarking Concurrent Code

```go
package main

import (
	"sync"
	"testing"
)

func BenchmarkChannelCommunication(b *testing.B) {
	ch := make(chan int)
	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		for i := 0; i < b.N; i++ {
			ch <- i
		}
		close(ch)
	}()

	go func() {
		defer wg.Done()
		for range ch {
		}
	}()

	wg.Wait()
}

func BenchmarkMutex(b *testing.B) {
	var mu sync.Mutex
	var counter int

	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			mu.Lock()
			counter++
			mu.Unlock()
		}
	})
}
```

Run with: `go test -bench=. -benchmem`

## Key Takeaways

1. **Use `WaitGroup` or `context`** to coordinate goroutine lifecycles
2. **Always provide cancellation** for long-running goroutines
3. **Channels are for communication**; mutexes are for protecting shared state
4. **Unbuffered channels provide synchronization**; buffered channels provide asynchrony
5. **Context is crucial** for request-scoped cancellation and deadlines
6. **Prevent goroutine leaks** by ensuring all goroutines can exit
7. **Use `errgroup`** for cleaner error handling in concurrent code
8. **Always test with `-race` flag** to catch data races
9. **Profile before optimizing** concurrent code
10. **Keep it simple**: Don't use concurrency if sequential code is sufficient
