# Go > An Intro to `if`, `switch`, and `for`

-   Unlike other programming languages, there is only `for` construct for looping through a list in Go:

```go
package main

import "fmt"

func main() {
	for i := 0; i < 100; i++ {
		fmt.Println(i)
	}
}
```

-   In order to loop infinitely, we have:

```go
package main

import "fmt"

func main() {
	fmt.Println("Welcome to GoBank!")

	for {
		var userInput int

		fmt.Println("Which operation you want to do?")
		fmt.Println("1. Deposit 2. Withdraw")
		fmt.Println("Please enter the number associated with the opertion you want to accomplish")
		fmt.Print("Your choice: ")

		fmt.Scan(&userInput)

		if userInput == 1 {
			fmt.Println("Ok, Now you can use our envelope-free service")
		} else if userInput == 2 {
			fmt.Println("Ok, How much money do you want to withdraw?")
		} else {
			fmt.Println("Opps, at the moment we cannot support your request")

			break
		}
	}

	fmt.Println("Goodbye!")
}
```

-   We can also use `return` instead of `break`; the problem though is that `fmt.Println("Goodbye!")` wont' be run anymore that way. To fix that we have:

```go
	for {
		var userInput int

		fmt.Println("Which operation you want to do?")
		fmt.Println("1. Deposit 2. Withdraw")
		fmt.Println("Please enter the number associated with the opertion you want to accomplish")
		fmt.Print("Your choice: ")

		fmt.Scan(&userInput)

		if userInput == 1 {
			fmt.Println("Ok, Now you can use our envelope-free service")
		} else if userInput == 2 {
			fmt.Println("Ok, How much money do you want to withdraw?")
		} else {
			fmt.Println("Opps, at the moment we cannot support your request")

			fmt.Println("Goodbye!")

			return
		}
	}
```

-   If we need to skip the rest of iteration commands and go for the next round, we can use the `continue` keyword
-   We can also do looping based on a condition as follows:

```go
func main() {
	count := 0

	for count < 100 {
		fmt.Println(count)
		count += 1
	}
}
```

## Looping through Slices

-   We can use the `for` loop in order to iterate over arrays, slices, and maps

```go
package main

import "fmt"

func main() {
	slice := []string{"Bez", "Far", "Dan"}

	for i := 0; i < len(slice); i++ {
		fmt.Println(slice[i])
	}
}
```

-   Another syntax for looping is `for range`:

```go
package main

import "fmt"

func main() {
	slice := []string{"Bez", "Far", "Dan"}

	for index, value := range slice {
		fmt.Println(index, value)
	}
}
```

-   If you do not care about individual indexes or values, you replace that value by underscore as in `for _, value := range slice`

## Looping through Maps

-   By the same token, we can loop through maps as follows:

```go
package main

import "fmt"

func main() {
	sites := map[string]string{"google": "http://google.com", "amazon": "http://aws.com"}

	for key, value := range sites {
		fmt.Println(key, value)
	}
}
```

-   In the output we have:

```text
google http://google.com
amazon http://aws.com
```

## Switch Statement

-   Contrary to other languages, in Go there is no need to add the `break` statement after each `case` because it will be done automatically for us

```go
package main

import "fmt"

func main() {
	fmt.Println("Welcome to GoBank!")

	for {
		var userInput int

		fmt.Println("Which operation you want to do?")
		fmt.Println("1. Deposit 2. Withdraw")
		fmt.Println("Please enter the number associated with the opertion you want to accomplish")
		fmt.Print("Your choice: ")

		fmt.Scan(&userInput)

		switch userInput {
		case 1:
			fmt.Println("Ok, Now you can use our envelope-free service")
		case 2:
			fmt.Println("Ok, How much money do you want to withdraw?")
		default:
			fmt.Println("Opps, at the moment we cannot support your request")
			fmt.Println("Goodbye!")

			return
		}

	}

}
```
- We can also rewrite the above `switch` statement as follows:
```go
switch {
case userInput == 1:
	fmt.Println("Ok, Now you can use our envelope-free service")
case userInput == 2:
	fmt.Println("Ok, How much money do you want to withdraw?")
default:
	fmt.Println("Opps, at the moment we cannot support your request")
	fmt.Println("Goodbye!")

	return
}
```
-   As shown above, in order to break out of outer loop while using the `switch` statement, we cannot use the `break` keyword; instead we must use the `return` keyword but if we replace the `switch` statement with `if`, the `break` keyword would work as expected

## How to Use `fallthrough` in `switch` Statements 
- There might be some specific use cases that no matter what the result of a `case`, we want the next `case` to run as well:
```go
switch {
case userInput == 1:
	fmt.Println("You chose 1")
	fallthrough
case userInput == 2:
	fmt.Println("You chose 2")
default:
	fmt.Println("N/A")
	return
}
```
- The `fallthrough` keyword lets us run the code for the next `case` even if the condition for that one is not met!


## How to Add Statements inside If Statement!

-   As shown below, the `if` statement includes an statement which makes our code base a little bit more concise

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	message, err := doSomething("tmp")

	if result := err; result != nil {
		fmt.Println("The error is:", err)

		return
	}

	fmt.Println(message)
}

func doSomething(name string) (string, error) {
	if name == "Bez" {
		return "Short of Behzad", nil
	}

	return "", errors.New("Not acceptable")
}
```

-   The important point about the above code is that the **Scope** of `result` is just inside the `if` statement and if we want to use the `result` variable outside that block, we will get compilation error
-   If we do not want to follow the above approach, we can simply refactor the above programs as follows:

```go
func main() {
	message, err := doSomething("tmp")
	result := err;

	if result != nil {
		fmt.Println("The error is:", err)

		return
	}

	fmt.Println(message)
}
```

## Comma OK Idiom

-   In Go, the "comma ok" idiom is a technique used with map types and channel operations to check for the presence of a value in a map or to safely receive data from a channel without causing a panic or deadlock.
-   Consider a scenario where you have a map and want to check if a particular key exists and retrieve its associated value. The "comma ok" idiom is commonly used in this context:

```go
package main

import "fmt"

func main() {
	timezones := map[string]string{"NY": "America/New_York", "LA": "America/Los_Angeles"}
	tz, ok := timezones["NY"]
	if ok {
		fmt.Println(tz)
	}
}

```

-   Here, `tz` will contain the value associated with the key "NY", and `ok` will be `true` if the key exists in the map. If the key doesn't exist, `ok` will be `false`.

-   We can also write the above program as follows to be more concise:

```go
package main

import "fmt"

func main() {
	timezones := map[string]string{"NY": "America/New_York", "LA": "America/Los_Angeles"}

	if tz, ok := timezones["NY"]; ok {
		fmt.Println(tz, ok)
	}
}
```

-   The "comma ok" idiom is also used when receiving data from channels
