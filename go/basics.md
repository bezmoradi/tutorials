# Go > Basics

-   Google create Go for three primary reasons:
    -   Efficient compilation
    -   Efficient execution
    -   Ease of coding
-   In Go, the `package main` declaration is a crucial part of creating executable programs. When you write a Go program, you organize your code into packages. A package is a collection of Go source files that work together to provide a set of functions. Each Go file starts with a `package` declaration, which defines the package to which the file belongs. The `package main` is a special package in Go. It indicates that the Go file is the starting point of the executable program. When you create a Go executable, it must have a `package main` declaration in the file that contains the `func main()`. The `func main()` serves as the entry point of the program, where execution begins when you run the compiled executable.
-   In the following program, `fmt` is one of built-in Go packages (Packages like `fmt` do not have a `func main()` just because they are not intended to be run as a program but used by other programs)

```go
package main

import "fmt"

func main() {
	fmt.Println("In the name of the most high")
}
```

-   Keep in mind that we can have only and only one `func main`
-   However, if we try to build an executable by running `go build`, we would get the following error:

```text
$ go build
go: go.mod file not found in current directory or any parent directory; see 'go help modules'
```

-   One module consists of multiple packages. In the above example, this sample project that we have created can be considered as a module and to make it a module, we need to run the following command:

```text
$ go mod init
go: cannot determine module path for source directory /Users/behzadmoradi/Documents/projects/goplayground (outside GOPATH, module path must be specified)

Example usage:
	'go mod init example.com/m' to initialize a v0 or v1 module
	'go mod init example.com/m/v2' to initialize a v2 module

Run 'go help mod init' for more information.
```

-   As shown above, we have gotten an error just because we have no specified a module name:

```text
$ go mod init hello-world-module
go: creating new go.mod: module hello-world-module
go: to add module requirements and sums:
	go mod tidy
```

-   Now a module called `hello-world-module` is successfully created and if we take a peek at the project directory, we would see a file called `go.mod` including the following is created:

```text
module hello-world-module

go 1.21.5
```

-   Now if we run `go build`, the program is successfully built and an executable file called `hello-world-module` will be added which by double clicking it, a new terminal window will be opened and program output is shown or as an alternative way, we can run it via terminal

```text
$ ./hello-world-module
In the name of the most high
```

-   The important note here is that for the `go build` command to run successfully, we have to have `package main` otherwise it will not figure out the entry point for our app.

## Split Code into Multiple Files

-   The following is fro Go docs:
    -   A set of files sharing the same PackageName form the implementation of a package. An implementation may require that all source files for a package inhabit the **same** directory.
-   It means that if we need to export some functions of the `main` package, they must be in the same folder; in other words, in the root of the project

```go
// project/external-function.go
package main

import "fmt"

func ExternalFunction() {
	fmt.Println("External function")
}

// project/main.go
package main

func main() {
	ExternalFunction()
}
```

-   We can spread functions in different files:

```go
// main.go
package main

func main() {
	externalFunction()
}
```

-   Also we have created another file called `utils.go`:

```go
// utils.go
package main

import (
	"fmt"
)

func externalFunction() {
	fmt.Println("External function called")
}
```

-   As both files belong to the same package (`main`), there is no need to import anything inside the `main.go` in order to use the external function. In order to run the app we have:

```text
$ go rum main.go utils.co
```

-   Or we can:

```text
$ go run .
```

## Split Code into Multiple Packages

-   In bigger projects, we can place functionalities into different packages
-   Every package **must** be placed inside its own subfolder in our project and that subfolder **must** have the same name as the package:

```text
.
├── go.mod
├── main.go
└── utils
    └── some-file-name.go
```

-   So we have:

```go
// some-file.go
package utils

import (
	"fmt"
)

func ExternalFunction() {
	fmt.Println("External function in another package called")
}

```

-   Now in order to import the package we have:

```go
package main

import "github.com/bez/go-practice/utils"

func main() {
	utils.ExternalFunction()
}

```

-   `github.com/bez/go-practice` is the name when we run `go mod init <some_name>` then we add the name of the package (`utils`) NOT the name of the file with `.go` and one other important thing is that only functions with the first Capital letter are public to the other packages. To get the name of the module, we can go to `go.mod` file:

```text
module github.com/bez/go-practice

go 1.21.5
```

-   With this rule in mind, if we want to keep a function private to a package, we can name it with first small letter naming convention

## How to Use Third-party Packages

-   To do so, we can use the `go get` command as follows:

```text
$ go get github.com/Pallinder/go-randomdata
go: downloading github.com/Pallinder/go-randomdata v1.2.0
go: added github.com/Pallinder/go-randomdata v1.2.0
```

-   Keep in mind the `https://` is not part of the path. Then if we take a peek at the `go.mod` we will have:

```text
module github.com/bez/go-practice

go 1.21.5

require github.com/Pallinder/go-randomdata v1.2.0 // indirect
```

-   Also a new file called `go.sum` will be automatically added which stores references to third-party packages that we have installed
-   Like Node.js that added `node_modules`, in Go we do not have such a thing but the target package will be downloaded globally on your system
-   If we share our code, other devs by running `go get` we can download all packages that are used inside our module.
-   Finally, we can use the package as follows:

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

-   In order to get a specific version of a package we have:

```text
$ go get github.com/Pallinder/go-randomdata@v1.2.3
```

-   And to get the latest version of a package we can do:

```text
$ go get github.com/Pallinder/go-randomdata@latest
```

## An Intro into `go install`

-   We we do `go install` inside the Go project, the binary for that project will be created and added to `$GOPATH/bin` folder
-   We can also use third-party packages and install them using the `go install` command. This way, the new package will be downloaded, built and copied into the `$GOPATH/bin` folder and in order to run it from anywhere on our machine, we need to give the full path as follows:

```text
$ ~/go/bin/<package_name>
```

## Alias

-   In case of naming conflicts with other packages or if we would like to name our package something other than the original name, we can use an Alias as follows:

```go
package main

import utilities "go-playground/utils"

func main() {
	utilities.ExternalFunction()
}
```

## An Intro to ASCII, Unicode, and UTF-8

-   Before digging deep into the above concepts, let's have a refresher on how computers work with 0s and 1s:

```text
+-------------+-------------+-------------------------+
|   1 Light   |     2^1     |   Represents 2 things   |
+-------------+-------------+-------------------------+
|   2 Light   |     2^2     |   Represents 4 things   |
+-------------+-------------+-------------------------+
|   3 Light   |     2^3     |   Represents 8 things   |
+-------------+-------------+-------------------------+
|   4 Light   |     2^4     |  Represents 16 things   |
+-------------+-------------+-------------------------+
|   5 Light   |     2^5     |  Represents 32 things   |
+-------------+-------------+-------------------------+
|   6 Light   |     2^6     |  Represents 64 things   |
+-------------+-------------+-------------------------+
|   7 Light   |     2^7     |  Represents 128 things  |
+-------------+-------------+-------------------------+
|   8 Light   |     2^8     |  Represents 256 things  |
+-------------+-------------+-------------------------+
```

-   In order to understand this concept better, let's imagine we have a light that an only have two states: On & Off. In other words, this one light can represents two states: On or Off. Now imagine instead of a light, we have two lights which in this case we can have four different states as follows:

```text
On On Off Off
Off Off On On
On Off On Off
Off on Off On
```

-   The binary representation of the above chart is as follows:

```text
1 1 0 0
0 0 1 1
1 0 1 0
0 1 0 1
```

-   ASCII uses 8 bits (1 byte) or in other words eight different 0s and 1s to represent different characters which in total becomes 256 different characters (The word "bit" is combined by "bi" of "binary" and "t" of "digit"). With this in mind, there is no wonder that a bit can hold two numbers: 0 & 1.
    As an example, the capital "A" is represented by `01000001`
-   The problem with ASCII was that it could not cover all the characters of all the languages around the world then Unicode was created which by using hexadecimal characters, could cover way more symbols
-   UTF-8 is a convenient way of storing Unicode characters and it can store up to 4 bytes (32 bits) of data like emojis etc

## Go by Example

-   https://gobyexample.com

## An Intro to The `go env` Command

-   By running the `go env` command inside the terminal, we'll get a bunch of metadata about our installation of Go:

```text
.
.
.
GOPATH='/Users/behzadmoradi/go'
GOROOT='/usr/local/go'
.
.
.
```

-   `GOROOT` is where the main Go binary is installed on our local machine
-   `GOPATH` is where the packages we download using the `go get` command are located
-   In other to go to either of the above paths, we can simply run `cd $GOPATH` inside terminal
