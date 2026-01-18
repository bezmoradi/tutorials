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

A thread is **software, not hardware**. It's a sequence of instructions that your operating system manages and executes. Think of it as a to-do list that tells the computer what to do step by step.

Every process has at least one thread (the main thread), but can have multiple threads running at the same time. Each thread executes code line by line.

**Threads vs CPU cores**: A CPU core is the actual physical hardware that runs code. Threads are just lists of instructions in memory. Your operating system takes these thread instructions and assigns them to CPU cores. You can have 100 threads but only 4 CPU cores—the OS just switches between threads very quickly, giving each one a turn on the available cores.

**Key analogy**: If a process is a restaurant kitchen, threads are the cooks working in that kitchen. They share the same kitchen space (memory) but can work on different tasks simultaneously.

### OS Threads vs. Threads in Different Contexts

Here's where it gets important for understanding Go:

**In traditional programming languages** (Java, C++, Python, C#):

When you create a "thread" in your code, you're creating an **OS thread** (operating system thread). These terms refer to the same thing. The operating system directly manages these threads, deciding when each one runs and on which CPU core.

Characteristics of OS threads:
- Created and managed by your operating system (Windows, macOS, Linux)
- Each OS thread requires significant memory, typically 1-2 MB just for its stack (where function calls and local variables are stored)
- Creating and switching between OS threads is relatively expensive because it involves the OS kernel
- You can't create thousands of OS threads without running into memory and performance problems

**In Go:**

Go introduces a different model. When you create a goroutine, you're **not** creating an OS thread. Instead:

- Go creates a small pool of actual OS threads behind the scenes (typically one per CPU core)
- Goroutines are "user-space threads" managed by Go's runtime scheduler, not the operating system
- Go's scheduler multiplexes many goroutines onto a few OS threads
- Each goroutine starts with only 2 KB of stack space (500x smaller than an OS thread)

This is why we call goroutines "lightweight threads". They're not managed by the OS, they use far less memory, and you can create millions of them without problems.

**The fundamental difference**:
- **Traditional languages**: 1 thread in your code = 1 OS thread (1-to-1 mapping)
- **Go**: Many goroutines are multiplexed onto few OS threads (M-to-N mapping)

Think of it like transportation:
- **OS threads** are like taxis—each person gets their own taxi (expensive, limited by how many you can afford)
- **Goroutines** are like a bus system—many passengers (goroutines) share a few buses (OS threads), efficiently managed by a dispatcher (Go's scheduler)

## What Is A Goroutine?

A goroutine is a lightweight thread managed by the Go runtime, and goroutines allow your program to run functions concurrently. Think of them as different tasks running simultaneously within a program, letting you handle multiple requests or perform various operations without waiting for each one to finish before starting the next. Unlike OS threads which typically require 1-2 MB of stack space, goroutines start with only 2 KB and can grow as needed. This means you can easily run thousands or even millions of goroutines in a single program, making Go exceptionally efficient for concurrent workloads.

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Println(runtime.NumGoroutine()) // Prints 1
}
```

At the very beginning of the `main` function, there won't be any additional goroutines created by your code, so typically, you'll see only one goroutine, which is the main goroutine running the `main` function itself.

## An Introduction to `WaitGroup`

`WaitGroup` from the `sync` package gives us the ability to create goroutines and wait for them to complete.

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func sayHi() {
	fmt.Println("Hi")
	wg.Done()
}

func sayBye() {
	fmt.Println("Bye")
	wg.Done()
}

func main() {
	wg.Add(2)

	go sayHi()
	go sayBye()

	wg.Wait()
}
```

In the above program, we have created an instance of `WaitGroup` on this line:

```go
var wg sync.WaitGroup
```

It's defined at the package level simply because we need to have access to it inside different functions. The way it works is that at the end of the `main` function we call `wg.Wait()` meaning the program must wait until all goroutines are done.

To turn function calls into goroutines, we prepend them with the `go` keyword. Since we have two goroutines that we need to wait for, at the very beginning of our `main` function we call `wg.Add(2)`, which tells Go to wait for 2 goroutines to complete.

As a last step, inside those functions we need to call `wg.Done()` to signal to the `WaitGroup` that the current function is done.

**Best practice**: Always call `wg.Add()` before launching the goroutine to avoid race conditions. Also, consider using `defer wg.Done()` as the first line in your goroutine to ensure it's always called even if the function panics.

The `runtime` package includes some useful functions to understand what's happening behind the scenes, and one of them is `NumGoroutine()`:

```go

func main() {
	wg.Add(2)

	go sayHi()
	go sayBye()

	fmt.Println(runtime.NumGoroutine()) // Prints 3

	wg.Wait()
}
```

If we execute our code, the `NumGoroutine` function prints 3 because we have a goroutine for the `main` function, one for the `sayHi` function, and one for the `sayBye` function.

The place we call `NumGoroutine` determines what the output will be. To experiment, let's change the above code as follows:

```go
func main() {
	wg.Add(2)

	fmt.Println(runtime.NumGoroutine()) // Prints 1

	go sayHi()
	go sayBye()

	wg.Wait()
}
```

Since we have one goroutine for the `main` function and `NumGoroutine` is called before launching the other goroutines, it outputs 1. As another experiment, let's update the code once more by placing the `NumGoroutine` call all the way to the bottom of the `main` function:

```go
func main() {
	wg.Add(2)

	go sayHi()
	go sayBye()

	wg.Wait()

	fmt.Println(runtime.NumGoroutine()) // Prints 1
}
```

It prints 1 because `wg.Wait()` waits for all custom-made goroutines to complete, then the next line is executed. Since we are still inside the `main` function and each Go program has at least one goroutine for that function, it outputs 1.

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

A mutex, short for "mutual exclusion," is a synchronization mechanism used in concurrent programming to ensure that only one thread or goroutine can access a shared resource or critical section at a time. The purpose of a mutex is to prevent data races and maintain the consistency of shared data.

### How a Mutex Works

**Locking**: When a goroutine wants to access a critical section or modify a shared resource, it must acquire the mutex by calling the Lock method. If the mutex is currently held by another goroutine, the requesting goroutine will be blocked until the mutex is released by the current holder.

**Critical Section**: Once a goroutine has acquired the mutex, it can safely access the shared resource or execute a critical section of code. The mutex ensures that only one goroutine is in this critical section at any given time.

**Unlocking**: After completing the critical section, the goroutine releases the mutex by calling the Unlock method. This allows other goroutines waiting for the mutex to acquire it and proceed.

### Understanding Race Conditions

In the context of Go, a race condition occurs when two or more goroutines access shared data concurrently, and at least one of them modifies the data. The behavior of the program becomes unpredictable because the outcome depends on the timing and order of execution of the goroutines.

Race conditions can lead to unexpected and erroneous behavior in a program. Go's runtime includes a race detector that helps identify such issues during development. The race detector can be enabled by using the `-race` flag when compiling or running a Go program. Here's a simple example of a race condition in Go:

```go
package main

import (
	"fmt"
	"sync"
)

var counter = 0
var wg sync.WaitGroup

func increment() {
	for i := 0; i < 1000; i++ {
		counter++
	}

	wg.Done()
}

func main() {
	wg.Add(2)

	go increment()
	go increment()

	wg.Wait()

	fmt.Println("Final Counter:", counter)
}
```

In the first run we might get `Final Counter: 1523`, in the second run `Final Counter: 1891`, and in the third run `Final Counter: 1645`. The value is unpredictable because two goroutines are concurrently incrementing a shared counter variable. Since there is no synchronization mechanism (like locks or channels), a race condition occurs, and the final value of the counter changes with each run.

To address this issue and avoid race conditions, you should use synchronization mechanisms like a mutex. Here's an example using a mutex:

```go
package main

import (
	"fmt"
	"sync"
)

var counter = 0
var wg sync.WaitGroup
var mutex sync.Mutex

func increment() {
	for i := 0; i < 1000; i++ {
		mutex.Lock()
		counter++
		mutex.Unlock()
	}

	wg.Done()
}

func main() {
	wg.Add(2)

	go increment()
	go increment()

	wg.Wait()

	fmt.Println("Final Counter:", counter)
}
```

In this modified version, `mutex` is used to ensure that only one goroutine can modify the `counter` variable at a time. The `Lock` and `Unlock` methods of the `mutex` guarantee the atomicity of the critical section, preventing race conditions and ensuring that the final value of counter is as expected. Now we will get `Final Counter: 2000` in the output without any race condition.

As an experiment, let's place the `mutex` variable inside the `increment` function:

```go
func increment() {
	var mutex sync.Mutex

	for i := 0; i < 1000; i++ {
		mutex.Lock()
		counter++
		mutex.Unlock()
	}
	wg.Done()
}
```

If we run the program, the race condition scenario is back. The reason is that by calling the `increment` function two times, we are creating two separate `mutex` variables—one for each function call. The locking and unlocking functionalities have no effect on each other. Each function call has its own mutex instance, so they don't actually coordinate access to the shared `counter` variable. This demonstrates why mutexes must be shared between goroutines to be effective.

## What Is Atomicity?

In the context of concurrent programming, atomicity refers to the property of an operation or a series of operations being executed as a single, indivisible unit. An atomic operation is one that appears to occur instantaneously from the perspective of other threads or processes, and it is not subject to interference by other concurrent operations.

In the context of shared variables, atomicity is crucial to prevent race conditions. A race condition occurs when multiple threads or goroutines access shared data concurrently, and at least one of them modifies the data. If the operations on the shared data are not atomic, the interleaving of operations from different threads can lead to unexpected and incorrect results.

By making the increment operation atomic, you ensure that it is executed as a single, uninterruptible unit. In Go, you can use synchronization mechanisms such as `mutex` or the `sync/atomic` package to achieve atomic operations.

The `sync/atomic` package provides atomic operations for basic types like integers, ensuring that operations like increments, decrements, swaps, etc., are atomic and free from race conditions.

We can rewrite the above program using the `sync/atomic` package as follows:

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var counter int32
var wg sync.WaitGroup

func increment() {

	for i := 0; i < 1000; i++ {
		atomic.AddInt32(&counter, 1)
	}

	wg.Done()
}

func main() {
	wg.Add(2)

	go increment()
	go increment()

	wg.Wait()

	fmt.Println("Final Counter:", counter)
}
```

Still there won't be any race condition and no matter how many times we run it, we'll get the same result: `Final Counter: 2000`.

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
