# Go > Basics

Google created Go for three primary reasons:

-   Efficient compilation
-   Efficient execution
-   Ease of coding

In Go, the `package main` declaration is a crucial part of creating executable programs. When you write a Go program, you organize your code into packages (A package is a collection of source files that work together to provide a set of functionalities). Each Go file starts with a `package` declaration, which defines the package to which the file belongs to. The `package main` is a special package in Go; it indicates that the Go file is the starting point of the executable program. When you create a Go executable, it must have a `package main` declaration in the file that contains the `main` function. This function serves as the entry point of the program, where execution begins when you run the compiled executable.  
In the following program, `fmt` is one of built-in Go packages (Packages like `fmt` do not have a `func main()` just because they are not intended to be run as a program but instead used by other programs):

```go
package main

import "fmt"

func main() {
	fmt.Println("In the name of the most high")
}
```

Keep in mind that we can have only and only one function called `main`. However, if we try to build an executable by running `go build`, we would get the following error:

```text
$ go build
go: go.mod file not found in current directory or any parent directory; see 'go help modules'
```

In the above example, this sample project that we have created can be considered as a module and to make it a module, we need to run the following command:

```text
$ go mod init
go: cannot determine module path for source directory /Users/<username>/Documents (outside GOPATH, module path must be specified)

Example usage:
	'go mod init example.com/m' to initialize a v0 or v1 module
	'go mod init example.com/m/v2' to initialize a v2 module

Run 'go help mod init' for more information.
```

As shown above, we have an error just because we have not specified a module name:

```text
$ go mod init hello-world-module
go: creating new go.mod: module hello-world-module
go: to add module requirements and sums:
	go mod tidy
```

Now a module called `hello-world-module` is successfully created and if we take a peek at the project directory, we would see a file called `go.mod` is created:

```text
module hello-world-module

go 1.21.5
```

Now if we run `go build`, the program is successfully built and an executable file called `hello-world-module` will be added which by double clicking it, a new terminal window will be opened and program output is shown or as an alternative way, we can run it via terminal:

```text
$ ./hello-world-module
In the name of the most high
```

The important note here is that for the `go build` command to run successfully, we have to have `package main` otherwise it will not figure out the entry point for our app.

## Split Code into Multiple Files

A set of files sharing the same package name form the implementation of a package. An implementation may require that all source files for a package inhabit the **same** directory. We can spread functions in different files:

```go
// main.go
package main

func main() {
	aFunction()
}
```

Also we have created another file called `utils.go`:

```go
// utils.go
package main

import (
	"fmt"
)

func aFunction() {
	fmt.Println("External function called")
}
```

As both files belong to the same package (`main`), there is no need to import anything inside the `main.go` in order to use the external function. In order to run the app we have:

```text
$ go rum main.go utils.co
```

Or we can:

```text
$ go run .
```

## Split Code into Multiple Packages

In bigger projects, we can place functionalities into different packages. Every package **must** be placed inside its own subfolder in our project and that subfolder should have the same name as the package:

```text
.
├── go.mod
├── main.go
└── utils
    └── some-file-name.go
```

So we have:

```go
// utils/some-file-name.go
package utils

import (
	"fmt"
)

func ExternalFunction() {
	fmt.Println("External function in another package called")
}
```

Now in order to import the package we have:

```go
package main

import "github.com/<username>/tutorial/utils"

func main() {
	utils.ExternalFunction()
}
```

In case of naming conflicts with other packages or if we would like to name our package something other than the original name, we can use an alias as follows:

```go
package main

import utilities "github.com/<username>/tutorial/utils"

func main() {
	utilities.ExternalFunction()
}
```

## How to Use Third-party Packages

To do so, we can use the `go get` command as follows:

```text
$ go get github.com/Pallinder/go-randomdata
go: downloading github.com/Pallinder/go-randomdata v1.2.0
go: added github.com/Pallinder/go-randomdata v1.2.0
```

Keep in mind the `https://` is not part of the path. Then if we take a peek at the `go.mod` we will have:

```text
module github.com/<username>/<project_name>

go 1.21.5

require github.com/Pallinder/go-randomdata v1.2.0 // indirect
```

Also a new file called `go.sum` will be automatically added which stores references to third-party packages that we have installed.  
Contrary to Node.js that has `node_modules`, in Go we do not have such a thing but the target package will be downloaded globally on your system. If we share our code, other devs can download all packages that are used inside our module by running `go get`. Finally, we can use the package as follows:

```go
package main

import (
	"fmt"

	"github.com/Pallinder/go-randomdata"
)

func main() {
	fmt.Println(randomdata.City())
}
```

In order to get a specific version of a package we have:

```text
$ go get github.com/Pallinder/go-randomdata@v1.2.3
```

And to get the latest version of a package we can do:

```text
$ go get github.com/Pallinder/go-randomdata@latest
```

## An Intro into `go install`

If we do `go install` inside a Go project' directory, the binary for that project will be created and added to `$GOPATH/bin` folder. We can also use third-party packages and install them using the `go install` command. This way, the new package will be downloaded, built and copied into the `$GOPATH/bin` folder and can be executed from anywhere on our machine.

## An Intro to The `go env` Command

By running the `go env` command inside the terminal, we'll get a bunch of metadata about our installation of Go:

```text
.
.
.
GOPATH='/Users/<username>/go'
GOROOT='/usr/local/go'
.
.
.
```

`GOROOT` is where the main Go binary is installed on our local machine and `GOPATH` is where the packages we download using the `go get` command are located.
