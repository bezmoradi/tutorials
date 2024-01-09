# Go > Map

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
