# Go > Maps

Map is a data structure in Go that can be used to group some values together (in other programming languages it's called object, hash table, dictionary, etc.). Basically, it can be thought of as a struct but slightly different. To understand how powerful maps are, let's first create a slice of strings and see what limitations we have while using that:

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

The problem with using a slice for storing some values is that the slice does not give us a hint about what we are dealing with; in other words, if somebody has no idea what those values are about, they would never figure it out. The other issue is that in order to get an element, we need to memorize its index which can be tricky. The map data structure can solve these issues:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
fmt.Println(goUseCases["cli"]) // Go to create fast and elegant CLIs.
```

Keep in mind that when the closing brace is on a new line, the trailing comma after the last key/value pair is mandatory, otherwise you will get a compile-time error. However, if the closing brace is on the same line as the last element, the trailing comma is optional (e.g., `map[string]int{"a": 1, "b": 2}` is valid).

The `map[string]` part of defining a map is to specify the type of the keys which in this case is a string and `string` is used to define the value type which in this case is also string (to illustrate further, suppose we require our values to be organized as a slice of strings. In this case, we can specify our map as `map[string][]string`). We can also initialize a map as empty then add elements to it:

```go
goUseCases := map[string]string{}
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
```

Maps are dynamic meaning you can always add elements to them or remove element from them:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
goUseCases["microservices"] = "It's possible to design gRPC microservices with Go"
fmt.Println(goUseCases["microservices"]) // It's possible to design gRPC microservices with Go
```

We can easily override a key/value pair by choosing an existing key and assigning a new value to it:

```go
goUseCases := map[string]string{
		"cloud":  "It's easier than ever to build cloud services with Go.",
		"cli":    "Go to create fast and elegant CLIs.",
		"web":    "Go powers fast and scalable web applications.",
		"devops": "Go is built to support both DevOps and SRE",
	}
goUseCases["web"] = "Updated value for the 'web' key"
fmt.Println(goUseCases["web"]) // Updated value for the 'web' key
```

You might guess creating maps using the `var` keyword is like this:

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

As maps are **not** automatically initialized, so you need to create the map using make or using a composite literal before you can use it. In other words, the above way of creating maps is completely wrong.

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

Unlike slices where `make()` can take 2 or 3 arguments (type, length, and optionally capacity), `make()` for maps takes only 2 arguments; the first one defines the type and the second one is a capacity hint (not length) that specifies the initial number of elements the map can hold before needing to grow:

```go
goUseCases := make(map[string]string, 4)
goUseCases["cloud"] = "It's easier than ever to build cloud services with Go."
goUseCases["cli"] = "Go to create fast and elegant CLIs."
goUseCases["web"] = "Go powers fast and scalable web applications."
goUseCases["devops"] = "Go is built to support both DevOps and SRE"
/*
	At this point memory reallocation is needed simply because
	we have initialized our map with the initial length of 4
	and the next assignment is exceeding that length
*/
goUseCases["microservices"] = "It's possible to design gRPC microservices with Go"
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

## Type Definitions for Maps

We can create custom type definitions for maps to make code more readable and to attach methods. Note that this creates a new type definition (not a type alias):

```go
// Type definition (creates a new distinct type)
type MyMap map[string]int

// Type alias (just an alternate name, requires '=')
// type MyMap = map[string]int
```

Here's how to use type definitions with maps:

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

As with `struct` types, we can add methods to custom maps as follows:

```go
package main

type golangUseCases map[string]string

func (g golangUseCases) add(key, value string) {
	g[key] = value
}

func main() {
	goUseCases := make(golangUseCases)
	goUseCases.add("cloud", "It's easier than ever to build cloud services with Go.")
	goUseCases.add("cli", "Go to create fast and elegant CLIs.")
	goUseCases.add("web", "Go powers fast and scalable web applications.")
	goUseCases.add("devops", "Go is built to support both DevOps and SRE")
}
```

Now it's time to write a test for the above newly-created method:

```go
package main

import "testing"

func TestAdd(t *testing.T) {
	key := "cli"
	value := "Go to create fast and elegant CLIs."
	goUseCases := golangUseCases{}
	goUseCases.add(key, value)
	if goUseCases[key] != value {
		t.Errorf("expected %v but got %v", value, goUseCases[key])
	}
}
```

## Comma OK Idiom with Maps

Accessing an element of a map returns two values instead of just one:

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

If the key exists, `ok` will be true; otherwise it will be false. We can refactor the above code as follows:

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

The comma ok idiom is particularly useful when you need to distinguish between a key that doesn't exist and a key that exists with a zero value:

```go
func main() {
	// Map with a zero value
	counts := map[string]int{
		"apples":  5,
		"bananas": 0, // Zero value, but key exists
	}

	// Check for "bananas"
	if count, ok := counts["bananas"]; ok {
		fmt.Printf("Bananas count: %d (key exists)\n", count) // Prints: Bananas count: 0 (key exists)
	}

	// Check for "oranges" (doesn't exist)
	if count, ok := counts["oranges"]; ok {
		fmt.Printf("Oranges count: %d\n", count)
	} else {
		fmt.Println("Oranges key does not exist") // This will print
	}
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

In Go, when you use the `for` loop with a range over a map, the iteration order is not guaranteed and is intentionally randomized. This behavior was introduced as a security feature to prevent attackers from crafting inputs that cause hash collisions, which could lead to denial-of-service attacks. The randomization also discourages developers from relying on any particular iteration order.

## Combine Other Data Structure with `map`

We can combine other DS with `map`. For example, in the following program a slice of strings is used as values:

```go
func main() {
	m := map[string][]string{}
	// Or this way
	// m := make(map[string][]string)

	m["foods"] = []string{"pizza", "noodles"}
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

## How to Embed A Struct in A Map

To do so we have:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p2 := person{
		firstName: "Jane",
		lastName:  "Doe",
	}

	m := map[string]person{
		p1.firstName: p1,
		p2.firstName: p2,
	}

	fmt.Println(m) // map[Jane:{Jane Doe} John:{John Doe}]
}
```

## Maps vs. Structs

There are some main fundamental differences between maps and structs.

### Maps:

-   Keys must be comparable types (types that support `==` and `!=`), which excludes slices, maps, and functions
-   All keys must be of the same type, and all values must be of the same type
-   We can iterate over all key/value pairs inside a map
-   Maps have reference semantics (when passed to functions, modifications are visible to the caller)
-   We can add/remove entries dynamically

### Structs:

-   Fields are pre-defined and we cannot dynamically add new fields
-   Different fields can have different types
-   Structs have value semantics (passed by value by default, but can also pass pointers to structs)
-   We cannot delete fields from a struct

## Map Internals and Advanced Concepts

Understanding how maps work under the hood helps you write more efficient code and avoid common pitfalls.

### How Go Implements Maps

Go's map implementation uses a hash table with buckets. When you create a map, Go allocates an array of buckets, where each bucket can hold up to 8 key-value pairs. Here's how it works:

1. **Hashing**: When you add a key-value pair, Go computes a hash of the key
2. **Bucket selection**: The hash determines which bucket stores the pair
3. **Storage**: The key-value pair is stored in the bucket
4. **Overflow buckets**: If a bucket fills up, Go creates overflow buckets linked to the original

```go
package main

import "fmt"

func main() {
	// When you create a map, Go allocates initial buckets
	m := make(map[string]int, 10)

	// As you add elements, they're distributed across buckets based on hash
	m["apple"] = 1
	m["banana"] = 2
	m["cherry"] = 3

	fmt.Println(m)
}
```

### Performance Characteristics

Maps have specific time and space complexity characteristics:

-   **Time complexity**:
    -   Average case: O(1) for insert, lookup, and delete
    -   Worst case: O(n) when all keys hash to the same bucket (rare with good hash functions)
-   **Space complexity**: O(n) where n is the number of key-value pairs, plus overhead for buckets

**Load factor and rehashing**: When a map grows beyond its capacity, Go automatically rehashes it (creates a new, larger bucket array and redistributes all entries). This is why preallocating with `make(map[K]V, hint)` improves performance when you know the approximate size:

```go
package main

import "fmt"

func main() {
	// Without size hint - may trigger multiple rehashes
	m1 := make(map[string]int)
	for i := 0; i < 10000; i++ {
		m1[fmt.Sprintf("key%d", i)] = i
	}

	// With size hint - avoids unnecessary rehashing
	m2 := make(map[string]int, 10000)
	for i := 0; i < 10000; i++ {
		m2[fmt.Sprintf("key%d", i)] = i
	}
}
```

### Memory Considerations

Maps have memory overhead beyond just storing keys and values:

-   Each bucket has metadata (hash bits, overflow pointers)
-   Empty buckets still consume memory
-   Deleting entries doesn't shrink the map's allocated memory

The following example demonstrates this behavior. Note that actual memory values may vary based on Go version, system architecture, and garbage collector activity:

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	var m runtime.MemStats

	// Check memory before
	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc before: %v KB\n", m.Alloc/1024)

	// Create large map
	bigMap := make(map[int]string, 1000000)
	for i := 0; i < 1000000; i++ {
		bigMap[i] = "value"
	}

	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc after creation: %v KB\n", m.Alloc/1024)

	// Delete all entries
	for k := range bigMap {
		delete(bigMap, k)
	}

	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc after deletion: %v KB\n", m.Alloc/1024)
	// Notice: Memory is still allocated even though map is empty

	// Memory is still allocated - to free it, set to nil
	bigMap = nil
	runtime.GC()

	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc after nil: %v KB\n", m.Alloc/1024)
}
```

**Key takeaway**: If you need to free memory from a large map, setting it to `nil` (and allowing garbage collection) is the only way to release the allocated memory. Simply deleting all entries keeps the underlying bucket array allocated.

### Concurrency and Thread Safety

**Maps are not thread-safe**. Concurrent reads and writes to the same map will cause a runtime panic. If multiple goroutines need to access a map, you must use synchronization:

```go
package main

import (
	"fmt"
	"sync"
)

// Approach 1: Using sync.RWMutex
type SafeMap struct {
	mu sync.RWMutex
	m  map[string]int
}

func (sm *SafeMap) Get(key string) (int, bool) {
	sm.mu.RLock()
	defer sm.mu.RUnlock()
	val, ok := sm.m[key]
	return val, ok
}

func (sm *SafeMap) Set(key string, value int) {
	sm.mu.Lock()
	defer sm.mu.Unlock()
	sm.m[key] = value
}

func main() {
	// Safe for concurrent use
	sm := SafeMap{m: make(map[string]int)}

	var wg sync.WaitGroup
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func(n int) {
			defer wg.Done()
			sm.Set(fmt.Sprintf("key%d", n), n)
		}(i)
	}
	wg.Wait()

	fmt.Printf("Map has %d entries\n", len(sm.m))
}
```

For specialized concurrent use cases, Go provides `sync.Map`:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	// sync.Map is optimized for two scenarios:
	// 1. When entries are written once and read many times
	// 2. When multiple goroutines read, write, and overwrite disjoint key sets
	var sm sync.Map

	// Store values
	sm.Store("key1", "value1")
	sm.Store("key2", "value2")

	// Load values
	if val, ok := sm.Load("key1"); ok {
		fmt.Println(val)
	}

	// Delete
	sm.Delete("key1")

	// Range over entries
	sm.Range(func(key, value interface{}) bool {
		fmt.Printf("%v: %v\n", key, value)
		return true // continue iteration
	})
}
```

### Why Map Iteration Order Is Randomized

Since Go 1.0, map iteration order is intentionally randomized as a security feature. This prevents attackers from crafting inputs that cause hash collisions, which could lead to denial-of-service attacks:

```go
package main

import "fmt"

func main() {
	m := map[string]int{
		"a": 1,
		"b": 2,
		"c": 3,
	}

	// Run this multiple times - order changes
	for k, v := range m {
		fmt.Printf("%s: %d\n", k, v)
	}
}
```

If you need deterministic order, sort the keys:

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	m := map[string]int{
		"zebra":  1,
		"apple":  2,
		"banana": 3,
	}

	// Get keys and sort them
	keys := make([]string, 0, len(m))
	for k := range m {
		keys = append(keys, k)
	}
	sort.Strings(keys)

	// Iterate in sorted order
	for _, k := range keys {
		fmt.Printf("%s: %d\n", k, m[k])
	}
}
```

## Map Edge Cases and Gotchas

### Nil Map vs Empty Map

A nil map and an empty map behave differently:

```go
package main

import "fmt"

func main() {
	// Nil map - not initialized
	var nilMap map[string]int
	fmt.Printf("nilMap == nil: %v\n", nilMap == nil)
	fmt.Printf("len(nilMap): %d\n", len(nilMap))

	// Reading from nil map is safe (returns zero value)
	val := nilMap["key"]
	fmt.Printf("Read from nil map: %v\n", val)

	// Writing to nil map causes panic!
	// nilMap["key"] = 1 // panic: assignment to entry in nil map

	// Empty map - initialized but empty
	emptyMap := make(map[string]int)
	fmt.Printf("emptyMap == nil: %v\n", emptyMap == nil)
	fmt.Printf("len(emptyMap): %d\n", len(emptyMap))

	// Writing to empty map works fine
	emptyMap["key"] = 1
	fmt.Printf("emptyMap after write: %v\n", emptyMap)
}
```

### Map Comparability

Maps are only comparable to `nil`. You cannot use `==` to compare two maps:

```go
package main

import "fmt"

func main() {
	m1 := map[string]int{"a": 1}
	m2 := map[string]int{"a": 1}

	// This compiles
	fmt.Println(m1 == nil)

	// This does not compile
	// fmt.Println(m1 == m2) // invalid operation: m1 == m2 (map can only be compared to nil)

	// To compare maps, you must iterate and compare each entry
	// Or use maps.Equal from the maps package (Go 1.21+)
}
```

### Key Type Requirements

Map keys must be comparable types (types that support `==` and `!=`). This includes:

-   **Valid key types**: booleans, numbers, strings, pointers, channels, interfaces, structs (if all fields are comparable), arrays (if elements are comparable)
-   **Invalid key types**: slices, maps, functions

```go
package main

import "fmt"

// Valid: struct with comparable fields
type Point struct {
	X, Y int
}

// Valid: array of comparable type
type Coords [2]int

func main() {
	// Valid: int keys
	m1 := map[int]string{1: "one", 2: "two"}

	// Valid: struct keys (all fields comparable)
	m2 := map[Point]string{
		{X: 0, Y: 0}: "origin",
		{X: 1, Y: 1}: "diagonal",
	}

	// Valid: array keys
	m3 := map[Coords]string{
		{0, 0}: "origin",
		{1, 1}: "diagonal",
	}

	fmt.Println(m1, m2, m3)

	// Invalid: slice keys (uncomment to see compile error)
	// m4 := map[[]int]string{} // invalid map key type []int
}
```

### Map Pointer Semantics

Maps are reference types - when you pass a map to a function, you're passing a reference to the underlying data structure, not a copy:

```go
package main

import "fmt"

func modifyMap(m map[string]int) {
	m["modified"] = 100
}

func main() {
	original := map[string]int{"initial": 1}
	fmt.Printf("Before: %v\n", original)

	modifyMap(original)

	// The original map is modified!
	fmt.Printf("After: %v\n", original)
}
```

This is different from structs and basic types which are passed by value. With maps, you don't need to return the map or use a pointer to have modifications visible to the caller - the map itself already has reference semantics.

## The `maps` Built-in Package

The [maps](https://pkg.go.dev/maps) package was introduced in Go 1.21 and significantly enhanced in Go 1.23. It provides type-safe utility functions for working with maps using generics. This package eliminates the need for writing repetitive map manipulation code.

### Cloning and Copying Maps

**Clone** creates a shallow copy of a map:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	original := map[string]int{
		"a": 1,
		"b": 2,
		"c": 3,
	}

	// Create a shallow clone
	cloned := maps.Clone(original)

	// Modify the clone
	cloned["d"] = 4
	delete(cloned, "a")

	fmt.Printf("Original: %v\n", original) // Original: map[a:1 b:2 c:3]
	fmt.Printf("Cloned: %v\n", cloned)     // Cloned: map[b:2 c:3 d:4]
}
```

**Copy** merges one map into another:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	dst := map[string]int{
		"a": 1,
		"b": 2,
	}

	src := map[string]int{
		"b": 20, // Will overwrite
		"c": 30,
	}

	// Copy all entries from src to dst
	maps.Copy(dst, src)

	fmt.Printf("dst after copy: %v\n", dst) // dst after copy: map[a:1 b:20 c:30]
}
```

### Comparing Maps

**Equal** checks if two maps have the same key-value pairs:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	m1 := map[string]int{"a": 1, "b": 2}
	m2 := map[string]int{"b": 2, "a": 1} // Same content, different order
	m3 := map[string]int{"a": 1, "b": 3} // Different value

	fmt.Printf("m1 == m2: %v\n", maps.Equal(m1, m2)) // true
	fmt.Printf("m1 == m3: %v\n", maps.Equal(m1, m3)) // false
}
```

**EqualFunc** allows custom comparison logic:

```go
package main

import (
	"fmt"
	"maps"
	"strings"
)

func main() {
	m1 := map[string]string{
		"name": "John",
		"city": "NYC",
	}

	m2 := map[string]string{
		"name": "JOHN",
		"city": "nyc",
	}

	// Case-insensitive comparison
	equal := maps.EqualFunc(m1, m2, func(v1, v2 string) bool {
		return strings.EqualFold(v1, v2)
	})

	fmt.Printf("Case-insensitive equal: %v\n", equal) // true
}
```

### Working with Map Entries

**DeleteFunc** removes entries based on a condition:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	m := map[string]int{
		"a": 1,
		"b": 2,
		"c": 3,
		"d": 4,
		"e": 5,
	}

	// Delete all entries with even values
	maps.DeleteFunc(m, func(k string, v int) bool {
		return v%2 == 0
	})

	fmt.Printf("After DeleteFunc: %v\n", m) // After DeleteFunc: map[a:1 c:3 e:5]
}
```

### Iterator Functions (Go 1.23+)

Go 1.23 introduced a new iteration protocol using iterators. An iterator is a function that yields values one at a time to a consumer function. This pattern allows for more composable and memory-efficient operations compared to materializing entire collections into slices.

The iterator pattern in Go 1.23 works as follows:
- An iterator is a function that takes a `yield` function as a parameter
- The iterator calls `yield` for each element it wants to produce
- If `yield` returns `false`, iteration stops early
- This allows for lazy evaluation and better performance in many scenarios

The maps package provides several iterator-based functions that leverage this pattern.

**All** returns an iterator over key-value pairs:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	m := map[string]int{
		"apple":  5,
		"banana": 3,
		"cherry": 8,
	}

	// Iterate using the All iterator
	for k, v := range maps.All(m) {
		fmt.Printf("%s: %d\n", k, v)
	}
}
```

**Keys** returns an iterator over map keys:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	m := map[string]int{
		"apple":  5,
		"banana": 3,
		"cherry": 8,
	}

	// Iterate over keys only
	for k := range maps.Keys(m) {
		fmt.Printf("Key: %s\n", k)
	}
}
```

**Values** returns an iterator over map values:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	m := map[string]int{
		"apple":  5,
		"banana": 3,
		"cherry": 8,
	}

	// Iterate over values only
	sum := 0
	for v := range maps.Values(m) {
		sum += v
	}
	fmt.Printf("Total: %d\n", sum) // Total: 16
}
```

### Collecting from Iterators

**Collect** creates a map from an iterator of key-value pairs:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	// Create a slice of key-value pairs
	pairs := []struct {
		key   string
		value int
	}{
		{"a", 1},
		{"b", 2},
		{"c", 3},
	}

	// Convert to an iterator function
	iter := func(yield func(string, int) bool) {
		for _, p := range pairs {
			if !yield(p.key, p.value) {
				return
			}
		}
	}

	// Collect into a map
	m := maps.Collect(iter)
	fmt.Printf("Collected map: %v\n", m) // Collected map: map[a:1 b:2 c:3]
}
```

**Insert** adds key-value pairs from an iterator to an existing map:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	existing := map[string]int{
		"a": 1,
		"b": 2,
	}

	// Create an iterator
	newPairs := func(yield func(string, int) bool) {
		yield("c", 3)
		yield("d", 4)
	}

	// Insert new pairs into existing map
	maps.Insert(existing, newPairs)
	fmt.Printf("After insert: %v\n", existing) // After insert: map[a:1 b:2 c:3 d:4]
}
```

### Practical Example: Merging Multiple Maps

Combining several maps package functions:

```go
package main

import (
	"fmt"
	"maps"
)

func main() {
	// Start with configuration defaults
	defaults := map[string]string{
		"theme":    "light",
		"language": "en",
		"timezone": "UTC",
	}

	// User preferences (override some defaults)
	userPrefs := map[string]string{
		"theme":    "dark",
		"fontSize": "14px",
	}

	// Environment-specific overrides
	envOverrides := map[string]string{
		"apiEndpoint": "https://api.example.com",
	}

	// Merge all configurations
	config := maps.Clone(defaults)
	maps.Copy(config, userPrefs)
	maps.Copy(config, envOverrides)

	fmt.Println("Final configuration:")
	for k, v := range config {
		fmt.Printf("  %s: %s\n", k, v)
	}

	// Remove any empty values
	maps.DeleteFunc(config, func(k, v string) bool {
		return v == ""
	})
}
```

### Performance Considerations

The maps package functions are well-optimized, but keep these points in mind:

-   **Clone** creates a shallow copy - if values are pointers, both maps reference the same objects
-   **Equal** has O(n) time complexity where n is the number of entries
-   **DeleteFunc** iterates through all entries, so use it wisely for large maps
-   Iterator functions (Go 1.23+) can be more memory-efficient than collecting all keys/values into slices

The maps package simplifies common operations and makes code more readable while maintaining good performance.
