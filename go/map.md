# Go > Maps

-   Map is a data structure in Go that can be be used to group some values together (In other programming languages it's called object, hash table, dictionary etc). Basically, it can be thought of as a struct but slightly different:

```go
package main

import "fmt"

func main() {
	goUseCases := []string{
		"It's easier than ever to build cloud services with Go.",
		"Go to create fast and elegant CLIs.",
		"Go powers fast and scalable web applications.",
		"Go is built to support both DevOps and SRE",
	}

	fmt.Println(goUseCases[1]) // Go to create fast and elegant CLIs.
}
```

-   The problem with using a slice for storing some values like use cases of Go language is that the slice does not give us a hint about what we are dealing with; in other words, if somebody has no idea what those features are, they wound never figure it out. The other issue is that in order to get an element, we need to memorize its index which can be tricky. The map data structure can solve these issues:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
fmt.Println(goUseCases["cli"]) // Go to create fast and elegant CLIs.
```

-   Keep in mind that like slices, the trailing command after the last key/value pair is mandatory otherwise you will get compile-time error.
-   The `map[string]` part of defining a map is to specify the type of the keys which in this case is a string and `string` is used to define the value which in this case is also string.
-   We can also initialized a map as empty then add elements to it:

```go
goUseCases := map[string]string{}
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

-   Maps in Go are dynamic meaning that your always can add or remove elements to them:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
goUseCases["microservices"] = "It's possible to desing gRPC microservices with go"
fmt.Println(goUseCases["microservices"]) // It's possible to desing gRPC microservices with go
```

-   If we choose an existing key and assign a new value to it, we can easily override the key/value pair
-   To delete a key we have:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
goUseCases["web"] = "Updated value fo the 'web' key"
fmt.Println(goUseCases["web"]) // Updated value fo the 'web' key
```

-   Creating maps using the `var` keyword is like this:

```go
var goUseCases map[string]string
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

-   If we try to run the above code though, we will get the following error:

```text
panic: assignment to entry in nil map
```

-   In Go, maps are **not** automatically initialized, so you need to create the map using make or using a composite literal before you can use it.

## How to Use the `make()` Function to Make Maps

-   As with slices, we can use the `make()` function to make maps. Let's first see maps in action without it:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
```

-   When we add a new key/value pair to the map, Go needs to allocate memory space to it but if we kind of know how many key/value pairs will be stored into our map, we can use `make()` in order to make our app a little bit more **efficient** due to not reallocating memory:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

-   Unlike slices that get three arguments, `make()` for maps only gets two argument; the first one defines the type and the second one defines the length.

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
/*
	At this point memory reallocation is needed simply because
	we have initialized our map with the initial length of 4
	and the next assignment is excceding that length
*/
goUseCases["microservices"] = "It's possible to desing gRPC microservices with go"
```

-   In the above example, for the first four key/pairs, Go does not need to do any reallocation but when we start adding the fifth one to our map, the reallocation process kicks in because we had specified reserved memory space for the first four key/pairs.

## How to Delete A Key from Maps

-   We can use the `delete` built-in function to delete keys from a map:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
delete(goUseCases, "cloud")
fmt.Println(goUseCases) // cloud key/value pair is gone
```

-   If we need to clear the whole map, we can use the build-in function `clear` as follows:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
clear(goUseCases)
fmt.Println(goUseCases) // map[]
```

## Add Type Alias

-   We can use aliases in order to make more readable code:

```go
package main

type golangUseCases map[string]string

func main() {
	goUseCases := make(golangUseCases)
	goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
	goUseCases["cli"] = "Go to create fast and elegant CLIs."
	goUseCases["web"] = "Go powers fast and scalable web applications."
	goUseCases["devops"] = "Go is built to support both DevOps and SRE"
}
```

-   If interested to use map literal instead of using the `make` function, we can refactor the above code like this:

```go
goUseCases := golangUseCases{}
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

## How to Check Key Existence?

-   Accessing an element of a map can return two values instead of just one:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
cli, ok := goUseCases["NON_EXISTING_KEY"]
if !ok {
	fmt.Println("No such key exists")
} else {
	fmt.Println(cli)
}
```

-   If the key exits, `ok` will be truthy; otherwise it will be falsy

## Maps vs. Structs

-   There some main fundamental differences between maps and structs. In maps, you can use anything as the key which give us more flexibility in comparison to structs. With structs, we have pre-defined keys and dynamically we cannot add new key/values pairs. Also we cannot delete a key from a struct (To learn more about structs, visit [Struct](https://github.com/bezmoradi/tutorials/blob/master/go/05-struct.md)).

## An Intro to `maps` Built-in Package

-   Package [maps](https://pkg.go.dev/maps) defines various functions useful with maps of any type.


# Old stuff

-   In Go, Map is a collection of key/value pairs (An Object equivalent in JS). All keys and values inside a Map must be of exactly the same type

```go
package main

import "fmt"

func main() {
	colors := map[string]string{"red": "F00", "green": "0F0", "blue": "00F"}

	fmt.Println(colors)
}
```

-   `map[string]string{}` says create a Map with keys as `string` and values as `string`
-   There are some other ways to define a Map in Go

```go
colors := make(map[string]string)

colors["red"] = "F00"
colors["green"] = "0F0"
colors["blue"] = "00F"
```

## How to Delete A Key

-   To delete a key, we can use `delete(colors, "blue")` statement. One thing to note here is that if the key does not exist, we won't get any runtime error

## Looping Through Maps

-   To iterate over a Map we have:

```go
func print(colors map[string]string) {
	for color, hex := range colors {
		fmt.Println(color, hex)
	}
}
```

-   In Go, when you use the for loop with a range over a map, the iteration order is not guaranteed and may appear random. This behavior is by design due to the way maps are implemented in Go. The rationale behind this design choice is to optimize for performance and efficiency. It allows Go's map implementation to remain lightweight and efficient for most operations without the overhead of maintaining a specific order. If you require a specific order while iterating over keys in a map, you would need to sort the keys explicitly or use another data structure that maintains order, like a slice of key-value pairs
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	m := map[string][]string{}

	m["foods"] = []string{"pizza", "noddles"}
	m["drinks"] = []string{"tea", "coke"}

	keys := make([]string, 0, len(m))

	for key := range m {
		keys = append(keys, key)
	}

	sort.Strings(keys)

	for _, key := range keys {
		fmt.Println(key, m[key])
	}
}
```

## Combine Other Data Structure with `map`

-   We can combine other DS with `map`. For example, in the following program a slice of strings is used for the values:

```go
func main() {
	m := map[string][]string{}
	// Or this way
	// m := make(map[string][]string)

	m["foods"] = []string{"pizza", "noddles"}
	m["drinks"] = []string{"tea", "coke"}

	fmt.Println(m)
}
```

## Comma OK Idiom

-   First, let's first see an example:

```go
package main

import "fmt"

func main() {
	companies := map[string]string{"Google": "https://google.com", "Amazon": "http:amazon.com"}

	delete(companies, "Google")

	value, ok := companies["Google"]

	if ok {
		fmt.Printf("The value for key 'Google' is %v\n", value)
	} else {
		fmt.Printf("The key `Google` does not exist")
	}
}
```

-   We can refactor the above program as follows:

```go
func main() {
	companies := map[string]string{"Google": "https://google.com", "Amazon": "http:amazon.com"}

	if value, ok := companies["Google"]; ok {
		fmt.Printf("The value for key 'Google' is %v\n", value)
	} else {
		fmt.Printf("The key `Google` does not exist")
	}
}
```

-   If we want to do early return, we can do so:

```go
func main() {
	companies := map[string]string{"Google": "https://google.com", "Amazon": "http:amazon.com"}

	var value, ok = "", false

	if value, ok = companies["Google"]; !ok {
		fmt.Printf("The key `Google` does not exist")
		return
	}

	fmt.Printf("The value for key 'Google' is %v\n", value)
}
```

## Exercise of Counting Words in A Phrase

-   The following program counts the number of repetition of words in a phrase using `map`:

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(countWords("In the name of the most high"))

}

func countWords(str string) map[string]int {
	c := map[string]int{}
	words := strings.Split(str, " ")

	for _, w := range words {
		if _, ok := c[w]; !ok {
			c[w] = 1
		} else {
			c[w]++
		}
	}

	return c
}
```

-   In the output we have:

```text
map[In:1 high:1 most:1 name:1 of:1 the:2]
```

## What Are Differences between Maps and Structs?

-   Maps:
    -   All keys & values must be of the same type
    -   We can iterate over all key/values pairs inside a Map
    -   A Map is a reference type
    -   We can add/remove properties to a Map
-   Structs:
    -   Both keys and values can be of different types
    -   Struct is a value type
    -   We have to clearly define all property types right off the bat
