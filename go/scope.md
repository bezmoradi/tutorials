# Go > Scope

-   Simply put, **Scope** is where a variable is declared and is accessible:

```go
package main

import "fmt"

var name string = "Go"

func main() {
	var name string = "Golang"
	fmt.Println(name) // Golang
}
```

-   When we have a variable with the same name as in the package scope, it's called Variable Shadowing

-   As shown above, we have defined two variables with identical names but different scopes meaning the `name` variable which is defined in package scope is both accessible in functions within that package and all other files that belong to the `package main`.
-   In the above program, the `fmt.Println()` function first check the function scope for a variable named `name`; if found, it would use that otherwise if would look for it at the package scope. Now if we change the `main()` function as follows:

```go
package main

import "fmt"

var name string = "Go"

func main() {
	fmt.Println(name) // Go
}
```

-   In the above case, the `Println` function first checks its surrounding scope which is the `main` function and as it does not find any `name` in that scope, it continues to the package scope and finally find a variable named `name`.

```go
// helper.go
package main

import "fmt"

func doSomething() {
	fmt.Println(name) // Go
}
```

-   Now we have created a file named `helper.go` which belongs to the `package main`; that's why whatever we have defined inside the `main.go` file is accessible inside this file as well. Now we can call the `doSomething()` function inside the `main()` function as follows:

```go
func main() {
	doSomething() // Go
}
```

## Variables Scoped to `if` Statements

-   As shown below, the variable `z` is only accessible within the `if` domain:

```go
package main

import "fmt"

func main() {
	if z := 10; z >= 10 {
		fmt.Println(z)
	}
	fmt.Println(z) // z isn't accessible outside the if statement
}
```

## Block-level Scope

-   If we need to be more restrictive, we can implement block-level scoping as follows;

```go
func main() {
	name := "Go"
	{
		name := "Golang"
		fmt.Println(name) // Golang
	}
	fmt.Println(name) // Go
}
```

## How to Define Block-level Functions

-   As functions are first-class citizens in Go, we can simply assign them to variables. In the following example, we have defined a function called `doSomething` inside the `main` function that is only and only accessible inside `main` and if we try to access is outside its scope, we will get `undefined: doSomething` error:

```go
package main

import "fmt"

func main() {
	doSomething := func() {
		fmt.Println("Doing something")
	}
	doSomething() // Doing something
}

func anotherFunction()  {
	doSomething() // undefined: doSomething
}
```
