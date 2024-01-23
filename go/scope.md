# Go > Scope

## Intro

Simply put, **Scope** is where a variable is declared and is accessible. In Go, we have three scope levels:

-   [Block-level Scope](#block-level-scope)
-   [Package-level Scope](#package-level-scope)
-   [App-level Scope](#app-level-scope)

## Block-level Scope

In the program that follows, the variable `name` is considered block-level as it is defined within the scope of the `main` function block:

```go
package main

import "fmt"

func main() {
	var name string = "Golang"
	fmt.Println(name) // Golang
}
```

When it comes to block-level scope, we can be even restrictive than the scope of a function block:

```go
func main() {
	{
		var name string = "Golang"
		fmt.Println(name) // Golang
	}

	fmt.Println(name) // undefined name
}
```

By add a pair of curly braces (`{}`), we can define a block inside another block and any variable defined inside that internal block is scoped only to that block and as shown in the above snippet, if we try to access `name` outside its block, we'll get `undefined name`.

A different form of block-level scope involves defining a variable within the scope of an `if` statement. As illustrated below, the variable `z` can only be accessed within the confines of the `if` scope:

```go
package main

import "fmt"

func main() {
	if z := 10; z >= 10 {
		fmt.Println(z)
	}
	fmt.Println(z) // undefined: z
}
```

If we try to compile our program, we'll get `undefined z` simply because `z` isn't accessible outside the `if` statement.

As functions are first-class citizens in Go, we can simply assign them to variables. In the following example, we have defined a function called `doSomething` inside the `main` function that is **only** accessible inside `main` and if we try to access is outside its scope, we will get `undefined: doSomething` error:

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

As `doSomething` is created inside the `main` function, we don't have access to it inside the `anotherFunction` function.

## Package-level Scope

If we define a variable in file outside any function, it's called package-level variable:

```go
package main

import "fmt"

var name string = "Go"

func main() {
	fmt.Println(name) // Go
}

func anotherFunction() {
	fmt.Println(name) // Go
}
```

In the above program, we have the `name` variable at package-level; that's why it's accessible from any other part of that package (It's not accessible from other packages though). Let's analyze what's happening when the `main` function is run.  
The go compiler fist checks the function scope of the `main` function to see whether it finds any variable called `name` or not which in this case there isn't any. Then it goes one level up and starts looking for the `name` variable at package-level.  
Now let's see what would happen if we have variables with identical names in different scopes:

```go
package main

import "fmt"

var name string = "Go"

func main() {
    var name string = "Golang"
	fmt.Println(name) // Golang
}
```

When we have a variable with the same name as in the package scope, it's called Variable Shadowing. As shown above, we have defined two variables with identical names but different scopes meaning the `name` variable which is defined in package scope is both accessible in functions within that package and all other files that belong to the `main` package.

In the above program, the `fmt.Println` function first checks the function scope for a variable called `name`; if found, it would use that otherwise if would look for it at the package scope. Let's change the `main` function as follows:

```go
package main

import "fmt"

var name string = "Go"

func main() {
	fmt.Println(name) // Go
}
```

In the above case, the `Println` function first checks its surrounding scope which is the `main` function and as it does not find any `name` variable in that scope, it continues to the package scope and finally finds a variable named `name`.

As mentioned earlier, variables defined within the package-level scope are accessible within that package, which implies they are also accessible in other files as long as those files belong to the same package:

```go
// helper.go
package main

import "fmt"

func doSomething() {
	fmt.Println(name) // Go
}
```

We have created a file named `helper.go` which belongs to the `main` package; that's why whatever we have defined inside the `main.go` file is accessible inside this file as well. To call the `doSomething` function inside the `main` function we can:

```go
func main() {
	doSomething() // Go
}
```

Consider that when our code is distributed across multiple files, it is necessary to specify all the file names in the following manner in order to run our program:

```text
$ go run main.go helper.go
```

It's clear that enumerating all file names, especially with a large number of files, can be quite cumbersome. As a workaround, we can employ the use of `.` instead:

```text
$ go run .
```

## App-level Scope

In Go, we have the option to access elements from other packages; let's call this feature App-level Scope. To see this feature in action, let's first see the project folder structure:

```text
.
├── go.mod
├── go.sum
├── main
│   └── main.go
└── utils
    └── utils.go
```

Basically, all files having to do with the `main` package are placed inside the `main` folder and by the same token, all files related to the `utils` package are created inside the `utils` folder. Let's first see the contents of the `utils.go` file:

```go
package utils

var Name string = "Go"
```

It only has a variable called `Name` (Pay attention to the casing, as this variable is defined with the first letter in uppercase). Inside the `main.go` file we have:

```go
package main

import (
	"fmt"
	"tutorials/utils"
)

func main() {
	fmt.Println(utils.Name) // Go
}
```

In order to use a variable which is defined within another package, first we need to import that package. But the important thing app-level scoping is the casing. In Go, if we want an entity be accessible outside its package-level scope, the first letter must be uppercase. For testing purposes, let's change the `utils.go` file as follows:

```go
package utils

var name string = "Go"
```

Then inside the `main` function we'll see:

```go
func main() {
	fmt.Println(utils.Name) // undefined utils.Name
}
```

Likewise, we can defined app-level functions:

```go
package utils

import "fmt"

var Name string = "Go"

func PrintSomething(input string) {
	fmt.Println(input)
}
```

As mentioned before, the name of the `PrintSomething` function must begin with an uppercase first letter. Then inside the `main` function we can call it this way:

```go
package main

import (
	"tutorials/utils"
)

func main() {
	utils.PrintSomething("Foo")
}
```

One final tip to know is that we can have identical variables within package-level and app-level scopes without any conflicts:

```go
package main

import (
	"fmt"
	"go-play/utils"
)

var Name string = "Golang"

func main() {
	fmt.Println(utils.Name) // Go
}
```

Within the package, we have defined a variable called `Name` and we also have a variable with the same name inside the `utils` package and they don't have any conflict at all.


## Exporting Variables from The `main` Package

In Go, the `main` package has a specific role; it's meant for executable programs. Other packages should not generally depend on the `main` package due to how the Go language is structured. Variables in the `main` package are accessible only within the `main` package itself and cannot be imported by other packages.



To read more about functions in Go, visit [Functions](https://github.com/bezmoradi/tutorials/blob/master/go/function.md)
