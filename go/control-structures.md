# Go > Control Structures

## `if` Statement

The following snippet is the simplest form of using the `if`, `else if`, and `else` statements:

```go
package main

import "fmt"

func main() {
	name := "Go"
	if name == "Go" {
		something := "This is only accessible inside the if block"
		fmt.Println("It's Go")
		fmt.Println(something)
	} else if name == "Golang" {
		fmt.Println("It's Golang")
	} else {
		fmt.Println("!")
	}
}
```

Unlike some other programming languages, Go does not require you to place any parenthesis around the `if` condition but if you do so, still it's valid code.  
As shown above, inside the `if` block we have defined a variable called `something`; this variable is only accessible within the `if` block.  
In Go, we have the option to place some statements in front of `if`. As an example, let's first see the normal version of writing conditional flow:

```go
package main

import (
	"fmt"
)

func isEven(number int) bool {
	return number%2 == 0
}

func main() {
	isEven := isEven(2)
	if !isEven {
		fmt.Println("It's odd")
	} else {
		fmt.Println("It's even")
	}
}
```

The `isEven` function returns `true` if the input param is even and `false` otherwise. Now let's refactor our code to make it a little bit more concise by placing the function call in front of the `if` statement:

```go
func main() {
	if isEven := isEven(2); !isEven {
		fmt.Println("It's odd")
		fmt.Printf("%v", isEven)
	} else {
		fmt.Println("It's even")
		fmt.Printf("%v", isEven)
	}
}
```

Unlike the previous example where `something` was accessible only inside the `if` block, the `isEven` variable is accessible within our control structure; in other words, it's accessible both inside the `if` and `else` blocks but still if we want to access it outside those blocks, we'll get compilation error.

### Comma OK Idiom

In Go, the "comma ok" idiom is a technique used with maps and channel operations to check for the presence of a value in a map or to safely receive data from a channel without causing a panic or deadlock. Consider a scenario where you have a map and want to check if a particular key exists and retrieve its associated value. The "comma ok" idiom is commonly used in this context:

```go
package main

import "fmt"

func main() {
	sites := map[string]string{
		"google": "https://google.com",
		"amazon": "https://amazon.com",
	}
	url, ok := sites["google"]
	if ok {
		fmt.Println(url)
	}
}
```

Here, `url` will contain the value associated with the key "google", and `ok` will be `true` because that key exists in the map. If the key doesn't exist though, `ok` will be `false`. We can also write the above `if` statement as follows to be more concise:

```go
if url, ok := sites["google"]; ok {
	fmt.Println(url)
}
```

### Comparing Composite Types

By composite types, we mean slices, maps, and structs. Contrary to most of other programming languages like JavaScript, in Go we do not have a `===` operator for deep equality; instead, the `==` operator is used for both shallow and deep equality depending on the context. For structs including primitive types like integers, floats, strings etc, we can use the traditional `if` statement for equality check:

```go
package main

import "fmt"

type person struct {
	name   string
	age    int
	isMale bool
}

func main() {
	p1 := person{
		name:   "John",
		age:    18,
		isMale: true,
	}
	p2 := person{
		name:   "John",
		age:    18,
		isMale: true,
	}

	if p1 == p2 {
		fmt.Println("equal")
	} else {
		fmt.Println("not equal")
	}
}
```

But Go does not allow direct comparison of structs if they contain slices, maps, or functions. To achieve deep equality in such cases, you'll need to use the `reflect` built-in package:

```go
package main

import (
	"fmt"
	"reflect"
)

type person struct {
	name            string
	age             int
	isMale          bool
	favoriteNumbers []int
}

func main() {
	p1 := person{
		name:            "John",
		age:             18,
		isMale:          true,
		favoriteNumbers: []int{1, 2, 3},
	}
	p2 := person{
		name:            "John",
		age:             18,
		isMale:          true,
		favoriteNumbers: []int{1, 2, 3},
	}

	if reflect.DeepEqual(p1, p2) {
		fmt.Println("equal")
	} else {
		fmt.Println("not equal")
	}
}
```

As our struct includes a key called `favoriteNumbers` which is a slice of integers, we have to use the `reflect` package for comparison.

## `for` Loop

Unlike other programming languages, there is only `for` construct for looping through an iterable in Go. As shown later, we can use the `for` loop in order to iterate over arrays, slices, and maps.

```go
package main

import "fmt"

func main() {
	for i := 0; i < 7; i++ {
		fmt.Print(i) // 0123456
	}
}
```

In most other languages we have the `while` loop to execute code until a condition is no longer true. In Go, we don't have such a loop and need to use the `for` loop instead:

```go
func main() {
	count := 0

	for count < 7 {
		fmt.Print(count)
		count += 1 // Or count++
	}
}

```

In the above case, `for count < 7` means "As long as `count` is less than 7". We can get the same result using an infinite loop like so:

```go
func main() {
	count := 0
	for {
		if count >= 7 {
			break
		}
		fmt.Print(count)
		count += 1
	}
}
```

As another example of an infinite loop we have:

```go
package main

import "fmt"

func main() {
	for {
		var input int
		fmt.Scan(&input)
		fmt.Println("input is:", input)
		if input == 10 {
			break
		}
	}
	fmt.Println("The end")
}
```

This Go code snippet defines a simple program using an infinite loop that repeatedly reads an integer input from the user, prints the input value, and breaks out of the loop if the input is equal to 10. We can also use `return` instead of `break`; the problem though is that `fmt.Println("The end")` wont' be run anymore that way. To fix that we have:

```go
func main() {
	for {
		var input int
		fmt.Scan(&input)
		fmt.Println("input is:", input)
		if input == 10 {
			fmt.Println("The end")
			return
		}
	}
}
```

If we need to skip the rest of iteration commands and go for the next round, we can use the `continue` keyword.  
We can use the `for` loop in order to iterate over arrays and slices:

```go
package main

import "fmt"

func main() {
	languages := []string{"Go", "Rust"}

	for i := 0; i < len(languages); i++ {
		fmt.Println(languages[i])
	}
}
```

Another syntax for looping is `for range`:

```go
func main() {
	languages := []string{"Go", "Rust"}

	for index, value := range languages {
		fmt.Println(index, value)
	}
}
```

If you do not care about individual indexes or values, you can replace them by an underscore as in `for _, value := range slice {}`. Likewise, we can loop through maps as follows:

```go
func main() {
	sites := map[string]string{
		"google": "https://google.com",
		"amazon": "https://amazon.com",
	}

	for key, value := range sites {
		fmt.Println(key, value)
	}
}
```

In the output we have:

```text
google https://google.com
amazon https://amazon.com
```

## Switch Statement

Earlier we saw how we can use the `else if` statement when we have more than one condition to evaluate; as another workaround, we can use the `switch` statement when we have multiple conditions to evaluate. Contrary to other languages, in Go there is no need to add the `break` statement after each `case` because it will be done automatically for us:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	switch time.Now().Weekday() {
	case time.Saturday:
		fmt.Println("It's weekend")
	case time.Sunday:
		fmt.Println("It's weekend")
	default:
		fmt.Println("It's a weekday")
	}
}
```

We can also rewrite the above `switch` statement as follows:

```go
func main() {
	switch {
	case time.Now().Weekday() == time.Saturday:
		fmt.Println("It's weekend")
	case time.Now().Weekday() == time.Sunday:
		fmt.Println("It's weekend")
	default:
		fmt.Println("It's a weekday")
	}
}
```

In the following program, in order to break out of the loop while using the `switch` statement, we cannot use the `break` keyword because it does not exit the program; instead we must use the `return` keyword:

```go
package main

import (
	"fmt"
)

func main() {
	for {
		var input int
		fmt.Scan(&input)

		switch input {
		case 1:
			fmt.Println("It's 1")
		case 2:
			fmt.Println("It's 2")
		default:
			return
		}
	}
}
```

But if we replace the `switch` statement with `if`, the `break` keyword would work as expected:

```go
func main() {
	for {
		var input int
		fmt.Scan(&input)

		if input == 1 {
			fmt.Println("It's 1")
		} else if input == 2 {
			fmt.Println("It's 2")
		} else {
			break
		}
	}
}
```

Inside the `else` block, instead of `break`, we can also use `return` to exit out of the program.
There might be some specific use cases that no matter what the result of a `case` is, we want the next `case` to run as well:

```go
package main

import (
	"fmt"
)

func main() {
	number := 1

	switch {
	case number == 1:
		fmt.Println("one")
		fallthrough
	case number == 2:
		fmt.Println("two")
	default:
		fmt.Println("N/A")
	}
}
```

The `fallthrough` keyword lets us run the code for the next `case` even if the condition for that one is not met!
