# Go > Maps

Map is a data structure in Go that can be be used to group some values together (In other programming languages it's called object, hash table, dictionary etc). Basically, it can be thought of as a struct but slightly different. To understand how powerful maps are, let's first create a slice of strings and see what limitations we have while using that:

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

The problem with using a slice for storing some values like use cases of the Go language is that the slice does not give us a hint about what we are dealing with; in other words, if somebody has no idea what those values are about, they would never figure it out. The other issue is that in order to get an element, we need to memorize its index which can be tricky. The map data structure can solve these issues:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
fmt.Println(goUseCases["cli"]) // Go to create fast and elegant CLIs.
```

Keep in mind that like slices, the trailing comma after the last key/value pair is mandatory otherwise you will get compile-time error.  
The `map[string]` part of defining a map is to specify the type of the keys which in this case is a string and `string` is used to define the value which in this case is also string (As another example, if we need out values be of type a slice of strings, we can define our map as `map[string][]string`). We can also initialized a map as empty then add elements to it:

```go
goUseCases := map[string]string{}
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

Maps in Go are dynamic meaning that you can always add elements to them or remove element from them:

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

We can easily override a key/value pair by choosing an existing key and assigning a new value to it:

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

Creating maps using the `var` keyword is like this:

```go
var goUseCases map[string]string
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

If we try to run the above code though, we will get the following error:

```text
panic: assignment to entry in nil map
```

In Go, maps are **not** automatically initialized, so you need to create the map using make or using a composite literal before you can use it. In other words, the above way of creating maps is completely wrong.

## How to Use the `make()` Function to Make Maps

As with slices, we can use the `make()` function to create maps. Let's first see maps in action without it:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
```

When we add a new key/value pair to the map, Go needs to allocate memory space to it but if we kind of know how many key/value pairs will be stored into our map, we can use `make()` in order to make our app a little bit more **efficient** due to not reallocating memory:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

Unlike slices that get three arguments, `make()` for maps only gets two argument; the first one defines the type and the second one defines the length.

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

In the above example, for the first four key/pairs, Go does not need to do any reallocation but when we start adding the fifth one to our map, the reallocation process kicks in because we had specified reserved memory space for the first four key/pairs.

## How to Delete A Key from Maps

We can use the `delete` built-in function to delete keys from a map:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
delete(goUseCases, "cloud")
```

If we need to clear the whole map, we can use the build-in function `clear` as follows:

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

We can use aliases in order to make more readable code:

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

If interested to use map literal instead of using the `make` function, we can refactor the above code like this:

```go
goUseCases := golangUseCases{}
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

## Comma OK Idiom with Maps

Accessing an element of a map can return two values instead of just one:

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

If the key exits, `ok` will be truthy; otherwise it will be falsy. We can refactor the above code as follows:

```go
func main() {
	goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}

	if cli, ok := goUseCases["NON_EXISTING_KEY"]; !ok {
		fmt.Println("No such key exists")
	} else {
		fmt.Println(cli)
	}
}
```

We can also do early returns like so:

```go
func main() {
	goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}

	var cli, ok = "", false
	if cli, ok = goUseCases["NON_EXISTING_KEY"]; !ok {
		fmt.Println("No such key exists")
		return
	}
	fmt.Println(cli)
}
```

## Looping Through Maps

-   To iterate over a Map we have:

```go
func main() {
	goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}

	for key, value := range goUseCases {
		fmt.Printf("%v -> %v\n", key, value)
	}
}
```

It outputs:

```text
cloud -> It's easier than ever to build cloud services with Go.
cli -> Go to create fast and elegant CLIs.
web -> Go powers fast and scalable web applications.
devops -> Go is built to support both DevOps and SRE
```

In Go, when you use the `for` loop with a range over a map, the iteration order is not guaranteed and may appear random. This behavior is by design due to the way maps are implemented in Go. The rationale behind this design choice is to optimize for performance and efficiency. It allows Go's map implementation to remain lightweight and efficient for most operations without the overhead of maintaining a specific order.

## Combine Other Data Structure with `map`

We can combine other DS with `map`. For example, in the following program a slice of strings is used as values:

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

## Exercise of Counting Words in A Phrase

The following program counts the number of repetition of words in a phrase using `map`:

```go
package main

import (
	"fmt"
	"strings"
)

func countWords(s string) map[string]int {
	words := strings.Split(s, " ")
	result := map[string]int{}

	for _, word := range words {
		word = strings.ToLower(word)
		if wordCount, ok := result[word]; ok {
			result[word] = wordCount + 1
		} else {
			result[word] = 1
		}
	}

	return result
}

func main() {
	fmt.Println(countWords("In the name of the most high"))
}
```

In the output we have:

```text
map[In:1 high:1 most:1 name:1 of:1 the:2]
```

Let's create a file called `main_test.go` and write some tests for the `countWords` function:

```go
package main

import "testing"

func TestCountWords(t *testing.T) {
	tests := []struct {
		name   string
		input  string
		result map[string]int
	}{
		{
			name:  "valid input",
			input: "this is to test with unique words",
			result: map[string]int{
				"this":   1,
				"is":     1,
				"to":     1,
				"test":   1,
				"with":   1,
				"unique": 1,
				"words":  1,
			},
		},
		{
			name:  "input with both uppercase and lowercase words",
			input: "word and Word and WORD and wORd are identical",
			result: map[string]int{
				"word":      4,
				"and":       3,
				"are":       1,
				"identical": 1,
			},
		},
		{
			name:   "empty string",
			input:  "",
			result: map[string]int{"": 1},
		},
	}

	for _, test := range tests {
		result := countWords(test.input)
		for key, expectedValue := range test.result {
			if resultValue, ok := result[key]; !ok || expectedValue != resultValue {
				t.Errorf("expected %v but received %v", test.result, result)
			}
		}
	}
}
```

## Maps vs. Structs

There are some main fundamental differences between maps and structs.

### Maps:

-   Keys can be anything
-   All keys & values must be of the same type
-   We can iterate over all key/values pairs inside a map
-   A map is a reference type
-   We can add/remove properties to a map

### Structs:

-   We have pre-defined keys and we cannot dynamically add new key/values pairs
-   Both keys and values can be of different types
-   Struct is a value type
-   We cannot delete a key from a struct

## An Intro to `maps` Built-in Package

Package [maps](https://pkg.go.dev/maps) defines various functions useful with maps of any type.
