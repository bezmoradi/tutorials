# Go > Functions

Most probably the simplest form of a function in Go is the one which doesn't accept any input params and also doesn't return anything:

```go
package main

import "fmt"

func doSomething() {
	fmt.Println("Something")
}

func main() {
	doSomething() // Something
}
```

In order for the above function to return something we have:

```go
func doSomething() string {
	return "Something"
}

func main() {
	fmt.Println(doSomething()) // Something
}
```

In order to receive function params we can:

```go
func doSomething(input string) string {
	return input
}

func main() {
	fmt.Println(doSomething("Something")) // Something
}
```

If input params are of the same type, we can write our function like so as well:

```go
func doSomething(input1, input2 string) string {
	return input1 + " " + input2
}

func main() {
	fmt.Println(doSomething("Something", "Something else")) // Something Something else
}
```

In Go, any function can return multiple values as follows:

```go
package main

import "fmt"

func doSomething(input1, input2 string) (string, string) {
	return input1, input2
}

func main() {
	firstReturnedValue, secondReturnedValue := doSomething("Something", "Something else")
	fmt.Println(firstReturnedValue)  // Something
	fmt.Println(secondReturnedValue) // Something else
}
```

Another syntax for returning multiple values is as follows:

```go
func doSomething(input1, input2 string) (returned1 string, returned2 string) {
	returned1 = input1
	returned2 = input2
	return
}

func main() {
	fmt.Println(doSomething("Something", "Something else")) // Something Something else
}
```

As shown above, we've given names to the return values then inside the function body assign whatever value we want to them then call the `return` statement without anything in front of. Meanwhile, if your want to be more explicit, you can return the values this way as well:

```go
func doSomething(input1, input2 string) (returned1 string, returned2 string) {
	returned1 = input1
	returned2 = input2
	return returned1, returned2
}
```

## Functions Accepting Any Kind of Values

In some situations, we might need to accept **any** input param. To do that we can:

```go
package main

import "fmt"

func printIt(input any) {
	fmt.Println(input)
}

func main() {
	printIt(7) // 7
}
```

By using the `any` type, we give a hint to Go compiler that any value is accepted. Instead of `any` we can also use the `interface{}` keyword:

```go
func printIt(input interface{}) {
	fmt.Println(input)
}
```

## Type Switch Syntax

`input.(type)` can only be used inside a `switch` statement and allows you to check what concrete type is stored in a variable of type `any`. In other words, it extracts the actual underlying type of whatever value is stored in `input`:

```go
package main

import "fmt"

func printIt(input any) {
	switch input.(type) {
	case int:
		fmt.Println("The input is an integer")
	case string:
		fmt.Println("The input is a string")
	default:
		fmt.Println("An undefined type")
	}
}

func main() {
	printIt(7)
}
```

The integer 7 gets stored in the `input` parameter (which has type `any`) then `input.(type)` reveals that the underlying type is `int` and finally the `switch` matches the `case int:` branch.

### Regular Type Assertions

Outside of `switch` statements, you'd use a different syntax for type assertions:

```go
func printIt(input any) {
	integerValue, ok := input.(int)
	if !ok {
		fmt.Println("It is not an integer")
		return
	}

	fmt.Printf("It is an integer of %v", integerValue) // It is an integer of 7
}
```
You can also combine the `if` statement with the type assertion as below:

```go
func print(input any) {
	if integerValue, ok := input.(int); ok {
		fmt.Printf("It is an integer of %v", integerValue)
		return
	}
	fmt.Println("It is not an integer")
}
```

## Functions as First-class Citizens in Go

As in JavaScript, functions in Go are first-class citizens meaning they can be assigned to variables, passed as arguments to other functions, and be returned as values from functions.

### Anonymous Functions

In the following example, we have assigned a function to a variable called `x` then called it:

```go
package main

import "fmt"

func main() {
	x := func() {
		fmt.Println("Anonymous function called")
	}
	x() // Anonymous function called
}
```

To create an anonymous function then call it right away, we have:

```go
func main() {
	func() {
		fmt.Println("An Anonymous function called")
	}() // An Anonymous function called
}
```

To add input params to an anonymous function we can:

```go
func main() {
	x := func(input string) {
		fmt.Println(input)
	}
	x("Anonymous function called")
}
```

Or if you want to call it right away you can:

```go
func main() {
	func(input string) {
		fmt.Println(input)
	}("Anonymous function called")
}
```

To have an anonymous function return some value, we can:

```go
func main() {
	x := func(input string) string {
		return input
	}
	y := x("Anonymous function called")
	fmt.Println(y) // Anonymous function called
}
```

To pass a function as a callback, we can:

```go
package main

import "fmt"

func add(a, b int) int {
	return a + b
}

func doMath(a int, b int, operation func(a, b int) int) int {
	return operation(a, b)
}

func main() {
	fmt.Println(doMath(6, 1, add)) // 7
}
```

Bear in mind the inside the `doMath` function, the type of the third argument(`operation`) is `func(a, b int) int` meaning that it must be a function which accepts two input params of type integers and returns an integer as well. Keep in mind that for function params, we can also remove the param names as follows:

```go
func doMath(a int, b int, operation func(int, int) int) int {
	return operation(a, b)
}
```

As another example we have:

```go
package main

import "fmt"

func transformNumbers(numbers []int, transformer func(int) int) []int {
	result := []int{}

	for _, value := range numbers {
		result = append(result, transformer(value))
	}

	return result
}

func double(number int) int {
	return number * 2
}

func triple(number int) int {
	return number * 3
}

func main() {
	numbers := []int{1, 2, 3, 4, 5, 6, 7}
	doubles := transformNumbers(numbers, double)
	triples := transformNumbers(numbers, triple)
	fmt.Println(doubles) // [2 4 6 8 10 12 14]
	fmt.Println(triples) // [3 6 9 12 15 18 21]
}
```

The `transformNumbers()` function accepts two parameters: a slice of integers and a function which accepts an input param of type integer and returns an integer (`func(int) int`). We can extract the function type into its own custom type to make our code cleaner:

```go
type transformerType func(int) int

func transformNumbers(numbers []int, transformer transformerType) []int {
	result := []int{}

	for _, value := range numbers {
		result = append(result, transformer(value))
	}

	return result
}
```

As long as the function that we pass as the second argument to the `transformNumbers()` function accepts an input of type integer and returns an integer, we are good to go (Both `double()` and `triple()` functions meed this criteria).

### Closures

Technically, when a function returns another function, it's called a **Closure**:

```go
package main

import "fmt"

func main() {
	fmt.Println(functionReturningAnotherFunction()()) // returned value
}

func functionReturningAnotherFunction() func() string {
	return func() string {
		return "returned value"
	}
}
```

Or to be more verbose, we can do it this way:

```go
func main() {
	functionReturningAValue := functionReturningAnotherFunction()
	fmt.Println(functionReturningAValue()) // returned value
}
```

As a more practical example we have:

```go
package main

import "fmt"

func main() {
	double := doMath(2)
	fmt.Println(double(10))
}

func doMath(outerParam int) func(int) int {
	return func(innerParam int) int {
		return outerParam * innerParam
	}
}
```

As shown above, the `doMath()` function is returning an anonymous function. The key point here is that the return type of the `doMath()` function must match the signature and return type of our inner anonymous function. We can rewrite the previous example where we needed `double` and `triple` function using closure:

```go
package main

import "fmt"

func doMath(factor int) func(number int) int {
	return func(number int) int {
		return factor * number
	}
}

func transformNumbers(numbers *[]int, transformer func(int) int) []int {
	doubles := []int{}

	for _, value := range *numbers {
		doubles = append(doubles, transformer(value))
	}

	return doubles
}

func main() {
	double := doMath(2)
	triple := doMath(3)

	numbers := []int{1, 2, 3, 4, 5, 6, 7}
	doubles := transformNumbers(&numbers, double)
	triples := transformNumbers(&numbers, triple)
	fmt.Println(doubles, triples)
}
```

## Recursion

If a function calls itself, it is called Recursion. A classic example for recursion is factorial. First, let's see how we can implement a factorial function without recursion:

```go
package main

import "fmt"

func main() {
	// Prints 120
	// Factorial of 5 = 5 * 4 * 3 * 2 * 1
	fmt.Println(factorial(5))
}

func factorial(number int) int {
	result := 1

	for i := 1; i <= number; i++ {
		result = result * i
	}

	return result
}
```

The recursive approach is as follows:

```go
func factorial(number int) int {
	if number == 0 {
		return 1
	}

	return number * factorial(number-1)
}
```

As a diagram, the above function with input param of 5 works like this:

```text
factorial(5) falsy condition because 5 > 0                        24 * 5 = 120
	factorial(4) falsy condition because 4 > 0                     6 * 4 = 24
		factorial(3) falsy condition because 3 > 0                 2 * 3 = 6
			factorial(2) falsy condition because 2 > 0             1 * 2 = 2
				factorial(1) falsy condition because 1 > 0         1 * 1 = 1
					factorial(0) TRUTHY condition because 0 == 0           1
```

## Variadic Functions

The `...` operator in the following function collects a list of stand-alone values and merge them all into a slice for us:

```go
package main

import "fmt"

func main() {
	fmt.Println(sumUp(1, 2, 3))
}

func sumUp(numbers ...int) int {
	sum := 0

	for i := 0; i < len(numbers); i++ {
		sum += numbers[i]
	}

	return sum
}
```

Such a function is called Variadic. Keep in mind that these functions can be called with zero arguments like `sumUp()`. As another example of such functions, we have the built-in `fmt.Println()` function which can be called with zero or more arguments. If need be, we can extract the few first values them accumulate the rest of then into the `numbers ...int`

```go
func sumUp(first int, numbers ...int) int {
	sum := 0

	for i := 0; i < len(numbers); i++ {
		sum += numbers[i]
	}

	return sum
}
```

In this case, 1 will be assigned to `first` which in the above example we have not used it then the rest of the values (2 and 3) will be stored inside the `numbers` slice. Now let's consider a scenario where we need to use the third-party function which accepts elements of a slice as stand-alone items but our data structure is a slice. In this case, we hove to do the opposite of the `...int` operation:

```go
func main() {
	slice := []int{1, 2, 3}
	fmt.Println(sumUp(slice...))
}
```

As shown above, this time around we need to pass the slice we have suffixed by `...`. In other words, behind the scenes `sumUp(slice...)` will be turned into `sumUp(1, 2, 3)`.  
As default function params is not built into the Go language, we can use the variadic params to build such a functionality:

```go
package main

import "fmt"

func greet(name string, greeting ...string) string {
	defaultGreeting := "Hi"
	if len(greeting) > 0 {
		defaultGreeting = greeting[0]
	}

	return defaultGreeting + " " + name

}

func main() {
	fmt.Println(greet("John"))                 // Hi John
	fmt.Println(greet("Jane", "Good morning")) // Good morning Jane
}
```

Basically, the second param of the `greet` function is set, it will be used otherwise the default value of `Hi` will be used.

## An Intro to The `defer` Keyword

The `defer` tells the compiler to do something later but before it finishes running the surrounding function which in the following case is the `main()` function:

```go
package main

import "fmt"

func main() {
	defer foo()
	bar()
}

func foo() {
	fmt.Println("foo")
}

func bar() {
	fmt.Println("bar")
}
```

In other words, we have:

```go
func main() {
	defer foo()
	bar()
	// foo() will be executed exactly at this point
}
```

To better figure it out we can add some logs as follows:

```go
func main() {
	fmt.Println("before foo() called")
	defer foo()
	fmt.Println("after foo() called")
	bar()
	fmt.Println("here is the last line")
}
```

And in the output have have:

```text
before foo() called
after foo() called
bar
here is the last line
foo
```

One of the main use cases of `defer` is to clean up resources. For example, when we open a file, connect to a database etc, we do not want to have those resources forever; instead we want resources to be removed when our current function is done. For example, when you repeatedly open files without closing them, you might eventually reach the limit of how many files can be open concurrently (which is often a limited number set by the operating system). When this limit is reached, attempting to open more files will fail, potentially causing your program to misbehave or crash due to the inability to access necessary resources. We also need to know that deferred functions run in LIFO order. For example:

```go
package main

import "fmt"

func main() {
	defer fmt.Println("3")
	defer fmt.Println("2")
	defer fmt.Println("1")
}
```

The last deferred function which prints `1` is the first function that will be executed; that's why we have:

```text
1
2
3
```

## An Intro to Wrapper Functions is Go

We can also create wrapper functions that accept another function and call it but before and after that function call, do some other operations (It can be compared to the Decorator Design Pattern). For example, in the following program we have created a wrapper function called `timer()` which calculated the total amount of time it takes to run its input function:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	timer(doSomeWork)
}

func doSomeWork() {
	for i := 0; i < 1_000; i++ {
		fmt.Println(i)
	}
}

func timer(f func()) {
	start := time.Now()

	f()

	elapsed := time.Since(start)

	fmt.Printf("Total time executing this function is: %v", elapsed)
}
```

## Inlining Decisions by Go Compiler

Inlining in the context of Go (or any other programming language) refers to a compiler optimization where a function's code is inserted directly into the place where it's called instead of being executed as a separate function. This can improve performance by reducing the overhead of function calls. When the compiler inlines a function, it's like copying and pasting its code wherever the function is used. It can make things faster because there's no need to jump to another part of the code to execute that function; everything happens right where it's needed. However, the decision of whether or not to inline a function is usually made by the compiler itself based on various factors like the size of the function, how many times it's called, and the optimization settings. Inlining can make your code faster, but it can also increase the size of the compiled code. It's a trade-off that the compiler balances based on what it thinks will be most efficient. As an example, let's dive deep into the following program:

```go
package main

import "fmt"

func add(a, b int) int {
	return a + b
}

func main() {
	x := 5
	y := 7

	result := add(x, y)

	fmt.Println("Result:", result)
}
```

When the compiler sees the `add` function being called inside the `main` function, it can choose to inline the `add` function. This means the `add` function's code will be directly placed inside the `main` function, instead of creating a separate execution context for it:

```go
func main() {
	x := 5
	y := 7
	result := x + y

	fmt.Println("Result:", result)
}
```

The addition operation `(x + y)` from the `add` function has been directly placed inside the `main` function, eliminating the need for a separate function call.
