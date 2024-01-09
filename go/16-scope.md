# Go > Scopes

-   Simply put, **Scope** is where a variable is declared and is accessible

```go
package main

import "fmt"

var name string = "Bez"

func main() {
    // When we have a variable with the same name
    // as in the package scope, it's called Variable Shadowing
	var name string = "Behzad"

	fmt.Println(name)
}
```

-   As shown above, we have defined two variables with identical names but different scopes meaning the `name` variable which is defined in package scope is both accessible in functions within that package and all other files that belong to the `package main`
-   In the above program, the `fmt.Println()` function first check the function scope for a variable named `name`; if found, it would use that otherwise if would look for it at the package scope. The output of the above program is `Behzad`
-   Now if we change the `main()` function as follows:

```go
func main() {
	fmt.Println(name)
}
```

-   This time around, we would see `Bez` in the terminal

```go
// helper.go
package main

import "fmt"

func doSomething() {
	fmt.Println(name)
}
```

-   Now we have created a file named `helper.go` which belongs to the `package main` that's why whatever we have inside the `main.go` file is accessible inside this file as well. Now we can call the `doSomething()` function inside the `main()` function as follows:

```go
package main

var name string = "Bez"

func main() {
	doSomething()
}
```

## Variables Scoped to `if` Statements

-   As shown below, the variable `z` is only accessible within the `if` domain

```go
package main

import "fmt"

func main() {

	if z := 10; z >= 10 {
		fmt.Println(z)
	}

	// z isn't accessible outside the if statement
	// fmt.Println(z)
}
```

## Block-level Scope

-   If we need to be more restrictive, we can implement block-level scoping as follows;

```go
package main

import "fmt"

func main() {
	name := "Behzad"

	{
		name := "Bez"

		fmt.Println("internal", name)
	}

	fmt.Println("external", name)
}
```
