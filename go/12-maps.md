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
