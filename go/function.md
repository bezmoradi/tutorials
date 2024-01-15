# Go > Functions

-   Let's first define a function then analyze it:

```go
package main

import "fmt"

func main() {
	fmt.Println(doSomething())
}

func doSomething() string {
	return "Something"
}
```

-   To define a custom function, we must always add the return type as well which in this case is `string`

## Parameters

-   In order to receive function params we can:

```go
package main

import "fmt"

func main() {
	fmt.Println(concatStrings("Bez", "Moradi"))
}

func concatStrings(l string, r string) string {
	return l + " " + r
}
```

-   If input params are of the same type, we can write our function like so as well:

```go
func concatStrings(l, r string) string {
	return l + " " + r
}
```

## Return Multiple Values

-   In Go, any function can return multiple values as follows:

```go
package main

import "fmt"

func main() {
	myName, myAge := returnMultipleValues("Behzad", 40)
	fmt.Println(myName, myAge)
}

func returnMultipleValues(name string, age int) (string, int) {
	return name, age
}
```

## Named Returned Values

-   Another syntax for returning multiple values is as follows:

```go
func main() {
	salary, age := returnMultipleValues(100000, 39)
	fmt.Println(salary, age)
}

func returnMultipleValues(salary, age int) (returnedSalary float64, returnedAge int) {
	returnedSalary = float64(salary) + 0.1
	returnedAge = age + 1

	return
}
```

-   As shown in `(returnedSalary float64, returnedAge int)`, we give a name to return values then inside the function body assign whatever value we want to them then call the `return` statement without anything in front of. If we want to be more explicit, we can so `return returnedSalary, returnedAge` as well.

## Scope

-   Any variable or constant which is defined inside a function like `main()` is **scoped** to that function and if we need them in other functions as well, we can define them as follows:

```go
package main

import "fmt"

var firstName, lastName = "Bez", "Moradi"

func main() {
	fmt.Println(concatStrings())
}

func concatStrings() string {
	return firstName + " " + lastName
}
```

-   From now on, the variables `firstName` and `lastName` are available to all functions inside this file
-   Keep in mind that if we need to define a constant or variable like so, we cannot use the shortcut for variable declaration like:

```go
firstName := "Bez"
```

-   The way we get compilation errors

## How to Create A Function Which Accepts Any Kind of Value

-   In some situation, we might need to accept **any** input param. To do that we have:

```go
package main

import "fmt"

func printIt(input interface{}) {
	fmt.Println(input)
}

func main() {
	printIt(12)
}
```

-   By using `interface{}`, we give a hint to Go compiler that any value is accepted. Instead of `interface{}` we can also use the `any` keyword

```go
func printIt(input any) {
	fmt.Println(input)
}
```

-   We can also combine the `any` type with `switch` in order to have full control on the user input

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
	printIt(12.1)
}
```

-   The only thing that you need to bear in mind is the special syntax of `input.(type)`
-   Another of getting the same functionality is through the following structure:

```go
func printIt(input any) {
	integerValue, ok := input.(int)

	if !ok {
		fmt.Println("It is not an integer")

		return
	}

	fmt.Printf("It is an integer of %v", integerValue)
}
```

-   The type `ok` is `bool`


## Functions as First-class Citizens in Go

-   As in JavaScript, functions in Go are first-class citizens meaning they can be assigned to variables, passed as arguments to other functions, and returned as values from functions.
-   In the following example, we have assigned a function to a variable called `x` the called it:

```go
package main

import "fmt"

func main() {
	x := func() {
		message := "An anonymous function without any params"
		fmt.Println(message)
	}

	x()

	y := func(message string) {
		fmt.Println(message)
	}

	y("An anonymous function with input param")
}

```

-   To return a function we have:

```go
package main

import "fmt"

func main() {
	value := functionReturningAnotherFunction()
	fmt.Println(value())
}

func functionReturningAnotherFunction() func() string {
	return func() string {
		return "returned value"
	}
}

```

-   To pass a function as callback :

```go
package main

import "fmt"

func main() {
	fmt.Println(doMath(10, 12, add))
}

func add(a, b int) int {
	return a + b
}

func doMath(a int, b int, operation func(a, b int) int) int {
	return operation(a, b)
}
```

-   As another example we have:

```go
package main

import "fmt"

func main() {
	numbers := []int{1, 2, 3, 4, 5, 6, 7}
	doubles := transformNumbers(&numbers, double)
	triples := transformNumbers(&numbers, triple)
	fmt.Println(doubles, triples)
}

func transformNumbers(numbers *[]int, transformer func(int) int) []int {
	doubles := []int{}

	for _, value := range *numbers {
		doubles = append(doubles, transformer(value))
	}

	return doubles
}

func double(number int) int {
	return number * 2
}

func triple(number int) int {
	return number * 3
}
```

-   The `transformNumbers()` function accepts two parameters: a slice of integers and a function which accepts an integer input param and returns an integer (`func(int) int`)
-   We can extract the function type into its own custom type to make our code cleaner:

```go
type transformerType func(int) int


func transformNumbers(numbers *[]int, transformer transformerType) []int {
	doubles := []int{}

	for _, value := range *numbers {
		doubles = append(doubles, transformer(value))
	}

	return doubles
}
```

-   As long as the function that we pass as the second argument to the `transformNumbers()` function accepts an input of type integer and returns an integer, we are good to go. Both `double()` and `triple()` functions meed this criteria

## Anonymous Functions

-   In the following program, we have specified three different anonymous functions: one without input param, one with input param, and yet another one with return type:

```go
package main

import "fmt"

func main() {
	func() {
		message := "An anonymous function without any params"
		fmt.Println(message)
	}()

	func(message string) {
		fmt.Println(message)
	}("An anonymous function with input param")

	fmt.Println(func() string {
		return "An anonymoud function with return type"
	}())
}
```

-   We can get the same result as the `double()` function but by using anonymous functions

```go
func main() {
	numbers := []int{1, 2, 3, 4, 5, 6, 7}
	doubles := transformNumbers(&numbers, func(number int) int {
		return number * 2
	})
	fmt.Println(doubles)
}
```

-   As seen above, the signature, body everything is the same as a normal function but it explicitly does not have any name. The main use case for an anonymous function is where another function needs a function as input param; in scenarios like this, we can define the function anonymously in place

## Closures

-   Another feature of a First-class Citizen Function is that it can return other functions which is a great use case for closures:

```go
package main

import "fmt"

type transformerType func(int) int

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

-   As shown above, the `doMath()` function is returning an anonymous function. The key point here is that that the return type of the `doMath()` function must match the signature and return type of our inner anonymous function

## Recursion

-   If a function calls itself, it is called Recursion. A classic example for recursion is factorial. First, let's see how we can implement a factorial function without recursion:

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

-   The recursive approach is as follows:

```go
func factorial(number int) int {
	if number == 0 {
		return 1
	}

	return number * factorial(number-1)
}
```

-   As diagram, the above function with input param of 5 works like this:

```text
factorial(5) Our condtion is falsy because 5 > 0                        24 * 5 = 120
	factorial(4) Our condtion is falsy because 4 > 0                     6 * 4 = 24
		factorial(3) Our condtion is falsy because 3 > 0                 2 * 3 = 6
			factorial(2) Our condtion is falsy because 2 > 0             1 * 2 = 2
				factorial(1) Our condtion is falsy because 1 > 0         1 * 1 = 1
					factorial(0) Our condtion is TRUTHY because 0 == 0   1
```

## Variadic Functions

-   The `...` operation in the following function collects the list of stand-alone values and merge them all into a slice for us

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

-   Such a function is called Variadic
-   Keep in mind the Variadic functions can be called with zero arguments like `sumeUp()`. An another example of such functions, we have the built-in `fmt.Println()` function which can be called with zero or more arguments
-   If need be, we can extract the few first values them accumulate the rest of them into the `numbers ...int`

```go
func sumUp(first int, numbers ...int) int {
	sum := 0

	for i := 0; i < len(numbers); i++ {
		sum += numbers[i]
	}

	return sum
}
```

-   In this case, 1 will be assigned to `first` which in the above example we have not used it then the rest of the values (2 and 3) will be stored into the `numbers` slice
-   Now let's consider a scenario the we need to use the third-party function which accepts elements of a slice as stand-alone items but our data structure is a slice. In this case, we hove to do the opposite of the `...int` operation

```go
func main() {
	slice := []int{1, 2, 3}
	fmt.Println(sumUp(slice...))
}
```

-   As shown above, this time around we need to pass the slice we have suffixed by `...`. In other words, behind the scenes `sumUp(slice...)` will be turned into `sumUp(1, 2, 3)`

## How to Use The `defer` Keyword with Function Calls

-   What `defer` keyword does is that is like telling the computer to do something later, but before it finishes running the function which in this case is `main()`. When you use `defer`, it schedules a function to be executed just before the function where defer was used in finishes running

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

-   In other words, we have:

```go
func main() {

	defer foo()
	bar()

	// foo() will be executed exactly at this point
}
```

-   To better figure it out we can add some logs as follows:

```go
func main() {
	fmt.Println("before foo() called")
	defer foo()
	fmt.Println("after foo() called")
	bar()
	fmt.Println("here is the last line")
}
```

-   And in the output have have:

```text
before foo() called
after foo() called
bar
here is the last line
foo
```

-   One of the main use cases of `defer` is to clean up resources. For example, when we open a file, connect to database etc, we do not want to have those resources forever; instead we want resources to be removed when our current function is done
-   For example, When you repeatedly open files without closing them, you might eventually reach the limit of how many files can be open concurrently (which is often a limited number set by the operating system). When this limit is reached, attempting to open more files will fail, potentially causing your program to misbehave or crash due to the inability to access necessary resources.

-   Deferred functions run in LIFO order. For example:

```go
package main

import "fmt"

func main() {
	defer fmt.Println("3")
	defer fmt.Println("2")
	defer fmt.Println("1")
}
```

-   The last deferred function has to print `1` to it is the first function that will be executed; that's why we have:

```text
1
2
3
```

## An Intro to Wrapper Functions is Go
- We can also create wrapper functions that accept another function and call it but before and after that function call, do some other operations. For example, in the following program we have created a wrapper function called `timer()` which calculated the total amount of time it takes to run its input function:
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