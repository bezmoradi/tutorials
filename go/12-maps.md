# Go > Maps

-   Map is a data structure in Go that can use be used to groups some values together. Basically, it can be thought of as a struct but slightly different:

```go
package main

import "fmt"

func main() {
	websites := []string{"https://google.com", "https://aws.com"}

	fmt.Println(websites[1])
}
```

-   The problem with using a slice for storing some values like website URLs is that the slice does not give us a hint about what we are dealing with; in other words, if somebody has no idea what AWS is, they wound never figure it out.
-   The other issue is that in order to get an element, we need to memorize its index which can be tricky somehow. The map data structure can solve these issues:

```go
package main

import "fmt"

func main() {
	websites := map[string]string{"google": "https://google.com", "amazon": "https://aws.com"}

	fmt.Println(websites["google"])
}
```

-   The `map[string]` part of defining a map is to specify the type of the keys which in this case is a string and `map[string]string` is used to define the value which in this case is also string. Maps in Go are dynamic meaning always can add or remove elements to them:

```go

func main() {
	websites := map[string]string{}
	websites["google"] = "https://google.com"
	websites["amazon"] = "https://aws.com"
	websites["linkedin"] = "https://linkedin.com"

	fmt.Println(websites)
}
```

-   As shown above, we can initialize an empty map then add key/value pairs to it
-   If we choose an existing key and assign a new value to it, we can easily override the key/value pair
-   To delete a key we have:

```go
func main() {
	websites := map[string]string{"google": "https://google.com", "amazon": "https://aws.com"}

	delete(websites, "google")

	fmt.Println(websites)
}
```

## Maps vs. Structs

-   There some main fundamental differences between maps and structs. In maps, you can use anything as the key which give us more flexibility in comparison to structs. With structs, we have pre-defined keys and dynamically we cannot add new key/values pairs. Also we cannot delete a key from a struct

## How to Use the `make()` Function with Maps

-   As with slices, we can use the `make()` function with maps. Let's first see maps in action without it:

```go
func main() {
	urls := map[string]string{}
	urls["google"] = "https://google.com"
}
```

-   When we add a new key/value pair to the map, Go needs to allocate memory space to it but if we kind of know how many key/value pairs will be stored into our map, we can use `make()` in order to make our app a little bit more efficient due to not reallocating memory

```go
func main() {
	urls := make(map[string]string, 10)
	urls["google"] = "https://google.com"
}
```

-   Unlike slices that get three arguments, `make()` for maps only gets two argument; the first one defines the type and the second one defines the overall length

```go
func main() {
	urls := make(map[string]string, 2)
	urls["google"] = "https://google.com"
	urls["amazon"] = "https://aws.com"
	urls["linkedin"] = "https://linkedin.com"

	fmt.Println(urls)
}
```

-   In the above example, for the first two key/pairs (`google` and `amazon`), Go does not need to do any reallocation but when we start adding the third, forth etc to our map, the reallocation process kicks in because we had specified reserved memory space for the first two key/pairs

## Add Type Alias
- We can use aliases in order to make a better DX
```go
package main

import "fmt"

type stringMap map[string]string

func (s stringMap) print() {
	fmt.Println(s)
}

func main() {
	urls := make(stringMap, 2) // Or urls := stringMap{}
	urls["google"] = "https://google.com"
	urls["amazon"] = "https://aws.com"
	urls["linkedin"] = "https://linkedin.com"

	urls.print()
}
```