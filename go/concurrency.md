# Go > Concurrency

-   Before looking into how concurrency works in Go, it's worth talking a little bit about dual core CPUs. In 2005, Intel released its first generation of dual core CPUs. Then in 2007, Google started working on Go and the language was officially announced to the public in March 2012, marking its initial release. Go, with its strong support for concurrency, could take advantage of multi-core architectures to efficiently handle parallel processing.

## Concurrency vs. Parallelism

-   Concurrency is about managing multiple tasks and making progress on them concurrently. It doesn't necessarily mean executing them simultaneously. It deals with the structure of the program and how tasks are organized. In a concurrent system, tasks can start, run, and complete in overlapping time periods, but they might not be actively executing at the same time.
-   Parallelism, on the other hand, involves actually executing multiple tasks simultaneously. It's about doing multiple things at the same time by leveraging multiple processors or cores. Parallelism is a way to achieve concurrency by physically running different tasks simultaneously
-   So if you have written a Go program that can handle different tasks concurrently but your code is run on a single core machine, no parallelism would happen!
-   Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan "Do not communicate by sharing memory; instead, share memory by communicating".

## What Is A Goroutine?

-   A goroutine is a lightweight lightweight threads managed by the Go runtime, and they allow your program to run functions concurrently. Think of them as different tasks running simultaneously within a program, letting you do stuff like handling multiple requests or performing various operations without waiting for each one to finish before starting the next.

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

-   At the very beginning of the `main` function, there won't be any additional goroutines created by your code, so typically, you'll see only one goroutine, which is the main goroutine running the `main` function itself.

## An Intro into `WaitGroup`

-   `WaitGroup` from the `sync` package gives us the ability to create some goroutines and wait for them to be completed.

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

-   In the above program, we have created an instance of `WaitGroup` on this line:

```go
var wg sync.WaitGroup
```

-   It's obvious that it's defines at the package level simply because we need to have access to it inside different functions. The way it works is that at the end of the `main` function we call `wg.Wait()` meaning the program must wait until all goroutines are done.
-   So in order to turn some functions calls to goroutines, we prepend them with the `go` keyword and as we at the moment we have two goroutines that we need to wait for, at the very beginning of our `main` function we call `wg.Add(2)` which simply means that Go needs to wait for 2 goroutines to get completed.
-   As a last step, inside those functions we need to call `wg.Done()` to give a hint to the `WaitGroup` that the current function is done.
-   The `runtime` package includes some useful functions to get to know what's happening behind the scenes and one them is `NumGoroutine()`:

```go

func main() {
	wg.Add(2)

	go sayHi()
	go sayBye()

	fmt.Println(runtime.NumGoroutine()) // Prints 3

	wg.Wait()
}
```

-   If we execute our code, the `NumGoroutine` function prints 3 because we have a goroutine for the `main` function, one for the `sayHi` function, and one for the `sayBye` function.
-   The place we call the `NumGoroutine` determines what the output will be! To experiment, let's change the above code as follows:

```go
func main() {
	wg.Add(2)

	fmt.Println(runtime.NumGoroutine()) // Prints 1

	go sayHi()
	go sayBye()

	wg.Wait()
}
```

-   As we have created one goroutine for the `main` function and the `NumGoroutine` is called right before other goroutines, it outputs 1. As another experiment, let's update the code once more by placing `NumGoroutine` call all the way to the bottom of the `main` function:

```go
func main() {
	wg.Add(2)

	go sayHi()
	go sayBye()

	wg.Wait()

	fmt.Println(runtime.NumGoroutine()) // Prints 1
}
```

-   It prints 1 because `wg.Wait()` waits for all custom-made goroutines then the next line is executed and as we are still inside the `main` function and each go program has at least one goroutine for that function, it outputs 1.

## Channels

-   Channels are blocking. In the following program, we are creating a channel called `c` which can hold any value of type `int` then in the next line are are adding `78` to our channel and this is the line where our channel blocks.

```go
package main

import "fmt"

func main() {
	c := make(chan int)

	c <- 78

	fmt.Println(<-c)
}
```

-   If we try to run the above code we will get:

```text
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:8 +0x38
exit status 2
```

-   This error occurs because you're trying to send a value into a channel but there's no goroutine (concurrent function) ready to receive it. In Go, sending and receiving from a channel can cause the program to wait until there's another goroutine ready to perform the complementary operation. In this case, when you create a channel with `c := make(chan int)`, it's an unbuffered channel, meaning it expects both a sender and a receiver to be ready simultaneously for communication. When you execute `c <- 78`, the main goroutine tries to send 78 into the channel, but since there's no other goroutine ready to receive from it, the program deadlocks. To fix this issue, you can do one of the following:
-   **Use Goroutines**: Create a separate goroutine to send the value into the channel while the main goroutine is waiting to receive it.

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

-   **Buffered Channel**: Create a buffered channel that can hold one value without requiring the sender and receiver to be ready simultaneously.

```go
package main

import "fmt"

func main() {
	c := make(chan int, 1)

	c <- 78

	fmt.Println(<-c)
}
```

-   Both of these solutions should prevent the deadlock and allow your program to execute without errors.
-   Channel can be made to specifically send values, receive values, or both.

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

-   This outputs:

```text
chan int
chan<- int
<-chan int
```

-   A more practical example of sending/receiving channels is as follows:

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

-   As channels are reference type, the same copy of `c` is sent to both functions. If we run the above program, it outputs:

```text
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.sender(...)
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:6
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:16 +0x3c
exit status 2
```

-   The reason being is due to the synchronization problem between the sender and receiver goroutines. When the sender function attempts to send a value to the channel `c <- 7`, it blocks because there's no other goroutine ready to receive the value from the channel at that moment. Similarly, when the receiver function attempts to receive a value from the channel `fmt.Println(<-c)`, it also blocks because there's no sender available to send the value.
-   To fix this, you need to run these functions concurrently using goroutines to avoid deadlocks. You can use the `go` keyword to launch these functions as goroutines:

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

-   It fixed the first issue; now we have another one though! Nothing is outputted in the terminal. The reason you don't see any output is that the `main` function finishes execution before the goroutines `sender` and `receiver` get a chance to execute due to the concurrent nature of goroutines. In Go, the `main` function **doesn't wait** for goroutines to finish by default. To fix that, we can either use `WaitGroup` of remove the `go` keyword behind the `receiver`. As the `receiver` function runs in the `main` goroutine, waits until a value is available on the channel. If a value is already there, it proceeds; otherwise, it blocks until a value is sent on the channel.

## How to Use Channels with Range

-   We can loop through a channel's value using a loop:

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

	fmt.Println("program existed successfully")
}
```

-   Technically, the `for` loop should iterate over all channel values until it's closed. Let's see what we get in the output:

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

-   In the above code, the `range` starts looping over the channel until the it's closed and the main problem here is that we have not explicitly asked our channel to get closed that'w why `range` is asking for more values whereas there is no more values and we get an error. To fix that, we need to close our channel when we are done:

```go
go func() {
	for i := 0; i < 10; i++ {
		c <- i
	}

	close(c)
}()
```

-   Now it outputs:

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

-   The `select` statement in Go is similar to a `switch` statement, but it's designed specifically for communication between goroutines via channels. While switch works with multiple expressions to determine the flow of control, select deals with communication operations on channels. In short:
    -   A select statement allows you to wait on multiple channel operations simultaneously.
    -   If none of the channels are ready, the select statement blocks until at least one of the channels is ready to proceed.

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

-   In the above program, The `select` statement listens to three channels: `even`, `odd`, and `quit`. When any of these channels are ready to be read from, the corresponding case inside the select will execute. If a value is received from the `even` channel, it prints it as an even number. If a value is received from the `odd` channel, it prints it as an odd number. And ff a value is received from the `quit` channel, it prints "Done" and exits the show function.
-   The `select` statement doesn't operate on values like a switch statement does. Instead, it chooses which channel operation to proceed with based on the readiness of the channels involved. It's a powerful construct for handling concurrent communication between goroutines.

## Another Example of Channels in Go

-   As function are called one after the other in order, if we have a long running task, it will block the flow of execution of other functions that come next:

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

-   We won't be able to see the returned value of the last `greet()` invocation right away because the previous function (`slowGreet()`) would take a while to complete
-   In order to tackle this issue and run all functions in parallel, we need to use the `go` keyword as follows:

```go
func main() {
	go greet("Nice to meet you")
	go greet("How are you?")
	go slowGreet("How ... are ... you ... ?")
	go greet("The final greeting")
}
```

-   But if we run the program now, we won't see any output in the terminal and the program exists right away! Technically, by adding the `go` keyword, you tell Go that you want to run those functions as Goroutines which simply means they will be run but this time in parallel instead one after the other
-   The idea behind running a function as a Goroutine is that it will be run in a non-blocking way; so the next function after can immediately be invoked
-   The reason that the program exits right away is that the `main()` function does not wait for Goroutines to get completed; in other words, what `main()` function does is that it dispatches the Goroutines and does not wait anymore
-   The solution to this problem is Go Channels. Simply put, a channel is a value that can be used as a communication channel when working with goroutines

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

-   This channel simply lets Go know whether the function process is done or not
-   In fact, the `slowGreet()` function when completes, sends a flag though its channel to the place where we started the goroutine which is the following line:

```go
go slowGreet("How ... are ... you ... ?")
```

-   Go will exit only after some data come out the channel on this line `<-done`
-   We can also place the returned value of the channel inside `fmt.Println(<-done)` or leave it as before
-   We can use a channel for completion of multiple goroutines. To do that, we have to make sure the channel is created before any goroutine and also we have to make sure that all goroutines accept such a channel

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

-   Keep in mind that we can use the same channel for multiple goroutines because our channel is a communication device and is designed in a way that can receive multiple values
-   However, if we run the above program, we will get a strange result:

```text
Hi Last call
```

-   Or in another execution we get:

```text
Hi Last call
Hi First call
Hi Second call
```

-   The reason why we are getting this unpredictable result is that we are facing race condition in our program where the function that completes first, ends the execution of the program. To tackle this issue we can:

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

-   In the output we have:

```text
Hi Last call
Hi Second call
Hi First call
Hi Delayed call
```

-   As seen above, the order of the output mismatches the order of function calls and the reason is that all those goroutines are executed in parallel and the output matches with the time those function invocations are done; in other words, there is possibility that the `greet()` function call with `Last call` finished way sooner than the function call with the `First call` input that's why that result set will be shown first
-   As a rule of thumb, if you use the same channel for multiple goroutines, you must wait for as many flags as you have goroutines
-   As this way of handling goroutines is not scalable and also error-prone (because you might forget to match the number of flags with the number of goroutines), there is an alternative approach as follows:

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

-   The above approach is better in comparison with the previous one, it is not ideal though that's why Go provides you with another simpler approach:

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

-   As shown above, we can directly loop though the channel which waits for all channels to complete but in the output we will get an error

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

-   We get this error because Go does not know when the channel is out of values; in other words, we loop through all values and wait for new values to be emitted but eventually there will be no value left and that is the reason we are getting this error

```go
func slowGreet(phrase string, doneChannel chan bool) {
	time.Sleep(3 * time.Second)
	fmt.Println("Hi", phrase)

	doneChannel <- true
	close(doneChannel)
}
```

-   Basically, by calling `close(doneChannel)` we would explicitly close a channel when we are done BUT when need to decided which function has the most delay and close the channel inside that function; This solution only works if you do know which operation takes longest and after that this channel is not needed anymore
-   One more thing is that if we do not care about the output of the channel, we can change the `for` loop inside the `main()` function as follows:

```go
for range done {
}
```

## Race Condition & Mutex

-   A mutex, short for "mutual exclusion," is a synchronization mechanism used in concurrent programming to ensure that only one thread or goroutine can access a shared resource or critical section at a time. The purpose of a mutex is to prevent data races and maintain the consistency of shared data. Here's a basic explanation of how a mutex works:

    -   **Locking**: When a goroutine wants to access a critical section or modify a shared resource, it must acquire the mutex by calling the Lock method. If the mutex is currently held by another goroutine, the requesting goroutine will be blocked until the mutex is released by the current holder.
    -   **Critical Section**: Once a goroutine has acquired the mutex, it can safely access the shared resource or execute a critical section of code. The mutex ensures that only one goroutine is in this critical section at any given time.
    -   **Unlocking**: After completing the critical section, the goroutine releases the mutex by calling the Unlock method. This allows other goroutines waiting for the mutex to acquire it and proceed.

-   In the context of Go, a race condition occurs when two or more goroutines (concurrent threads of execution in Go) access shared data concurrently, and at least one of them modifies the data. The behavior of the program becomes unpredictable because the outcome depends on the timing and order of execution of the goroutines.
-   Race conditions can lead to unexpected and erroneous behavior in a program. Go's runtime includes a race detector that helps identify such issues during development. The race detector can be enabled by using the `-race` flag when compiling or running a Go program. Here's a simple example of a race condition in Go:

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

-   In the first run we will get `Final Counter: 1395` and in the second one something unpredictable like `Final Counter: 1395`. In this example, two goroutines are concurrently incrementing a shared counter variable. Since there is no synchronization mechanism (like locks or channels), a race condition occurs, and the final value of the counter is unpredictable.
-   To address this issue and avoid race conditions, you should use synchronization mechanisms like a mutex. Here's an example using a mutex:

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

-   In this modified version, `mutex` is used to ensure that only one goroutine can modify the `counter` variable at a time. The `Lock` and `Unlock` functions of the `mutex` guarantee the atomicity of the critical section, preventing race conditions and ensuring that the final value of counter is as expected. Now we will get `Final Counter: 2000` in the output without any race condition.
-   As an experiment, let's place the `mutex` variable inside the `increment` function:

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

-   If we run the program, the race condition scenario is back into play! The reason being is that by calling the `increment` function two times, we are creating two variables called `mutex` for each that's why the locking and unlocking functionalities have no effect on each other; in other words, the `mutex` locks the `counter` variable then unlocks it in each iteration of the loop when we call the `increment` function for the first time and when in the second function call, we would have a totally different instance of `sync.Mutex` and for this reason mutating the `counter` variable becomes out of control.

## What Is Atomicity?

-   In the context of concurrent programming, atomicity refers to the property of an operation or a series of operations being executed as a single, indivisible unit. An atomic operation is one that appears to occur instantaneously from the perspective of other threads or processes, and it is not subject to interference by other concurrent operations.
-   In the context of shared variables, atomicity is crucial to prevent race conditions. A race condition occurs when multiple threads or goroutines access shared data concurrently, and at least one of them modifies the data. If the operations on the shared data are not atomic, the interleaving of operations from different threads can lead to unexpected and incorrect results.
-   By making the increment operation atomic, you ensure that it is executed as a single, uninterruptible unit. In Go, you can use synchronization mechanisms such as `mutex` or the `sync/atomic` package to achieve atomic operations.
-   The `sync/atomic` package provides atomic operations for basic types like integers, ensuring that operations like increments, decrements, swaps, etc., are atomic and free from race conditions.
-   We rewrite the above program using the `sync.atomic` package as follows:

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

-   Still the won't be any race condition and no matter how many times we run it, we'll get the same result.

## What Are The Differences between `sync.Mutex` And `sync.Atomic`?

-   Both `sync.Mutex` and `sync.atomic` package can be used to achieve synchronization and prevent race conditions in concurrent programming. However, there are differences between them, and the choice between them depends on the specific requirements of your program. Here are some key differences:
-   Mutex:
    -   Provides a way to protect a block of code (critical section) with a lock and unlock mechanism. It is suitable for protecting larger sections of code where multiple operations need to be performed atomically.
    -   Involves more overhead because it requires acquiring and releasing locks, and it may involve context switching between goroutines.
-   `sync/atomic`:
    -   Provides atomic operations on basic types (e.g., integers). It is more suitable for fine-grained operations, such as increments, swaps, and compare-and-swap operations on individual variables.
    -   Generally has lower overhead because it directly operates on memory without acquiring locks.
- In summary, if you need to protect larger sections of code or perform more complex synchronization, a mutex is often a suitable choice. If you have specific atomic operations on individual variables and need lower-level, more lightweight synchronization, the sync/atomic package might be more appropriate. 



## Links
https://www.youtube.com/watch?v=VkGQFFl66X4&list=PLKOhSssLmvQQPFv-QqbRZ10rn3UdBLvVC&index=3