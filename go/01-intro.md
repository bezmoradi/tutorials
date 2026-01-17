# Go > Introduction

For decades, single-threaded programs benefited from steady increases in CPU clock speeds. Based on trends before 2003, industry projections anticipated 10 GHz processors by 2005. However, fundamental physical constraints prevented this trajectory from continuing! Around 2004-2005, chip manufacturers encountered what became known as the "power wall". Increasing clock frequency requires higher voltage, and power consumption scales cubically with voltage. This relationship produces heat that exceeds practical cooling capabilities. As an example, Intel cancelled its planned 10 GHz "Tejas" processor in 2004, and clock speeds plateaued around 4 GHz.

The industry responded by increasing **core count** rather than clock speed. AMD introduced multi-core processors in 2004, followed by Intel in 2005. In March 2005, Herb Sutter published an influential article titled "The Free Lunch Is Over", arguing that developers could no longer rely on hardware improvements for performance gains and would need to write concurrent code to utilize multiple cores. However, concurrent programming with existing languages remained complex and error-prone. In September 2007, Robert Griesemer, Rob Pike, and Ken Thompson began designing a new language at Google with the explicit goal of making concurrent programming simpler and safer. That language became [Go](https://go.dev/).

## Installing Go on macOS

Before we start coding, let's install Go on macOS using Homebrew:

```sh
brew install go
```

Verify the installation by running:

```sh
go version
```

You should see output like: `go version go1.25.6 darwin/arm64` (your version may vary). To update Go to the latest version, we have:

```sh
brew upgrade go
```

## Hello World Go Program

Before writing our first Go program, it's important to develop a high-level understanding of Go's package and module concepts.

### What Are Packages?

A **package** is a collection of Go source files in the same directory that work together. Packages are Go's way of organizing and reusing code. Without packages, all code would be in one giant file, making it impossible to organize large projects or share code between projects. Packages solve this by providing namespaces and encapsulation.

In Go, `package main` has a special meaning; it tells Go that this package should be compiled into an executable program, not a library.

**Key rules**:

-   Executable programs must have exactly one `package main`
-   The `main` package must contain exactly one `main()` function
-   The `main()` function is the entry point where program execution begins
-   You cannot have multiple `main()` functions in your program
-   Packages like `fmt` don't have a `main()` function because they're libraries meant to be used by other programs, not run directly.

### What Are Modules?

Before Go 1.11, Go used `$GOPATH` for managing dependencies. Since Go 1.11, Go uses **modules** which is a collection of related packages with versioning support. Modules solve dependency management, version control, and reproducible builds. They let you specify exactly which version of each dependency your project uses.

With these concepts in mind, now let's start creating our first program.

```sh
mkdir hello-world && cd hello-world
```

Create a file named `main.go` with the following content:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

If we try to build an executable by running `go build` without a module, we would get an error:

```text
go: go.mod file not found in current directory or any parent directory; see 'go help modules'
```

This error tells us we need to create a module. Let's do that:

```sh
go mod init hello-world
```

We will have:

```text
go: creating new go.mod: module hello-world
go: to add module requirements and sums:
	go mod tidy
```

For local projects, you can use simple names like `hello-world`. For projects you plan to publish, use a path like `github.com/username/project`. Now a file called `go.mod` is created in your directory:

```text
module hello-world

go 1.25.6
```

Understanding `go.mod` starts with its structure. The first line, `module hello-world`, defines your module's name or import path and is used by Go to identify the project. The next line, `go 1.25.6`, specifies the minimum Go version required to build the module. Any additional lines that appear as you add dependencies list the external packages your project relies on, along with the exact versions Go should use.

Now if we run `go build`, the program compiles successfully:

```sh
go build
```

This creates an executable file named `hello-world`. Run it:

```sh
./hello-world
```

You'll see `Hello, World!`. The `go build` command requires `package main` to create an executable. Without it, Go doesn't know where the program should start. We can also get the output without first building the program:

```sh
go run main.go
```

**What just happened?** Go compiled your code to a temporary executable and ran it. For development, this is convenient. For production, you'll want to build a standalone executable.

## Organizing Code

### Multiple Files in One Package

As your code grows, you'll want to split it across multiple files for better organization. All files in the same directory with the same `package` declaration belong to that package. Let's split our code into two files. `main.go` will be as follows:

```go
package main

func main() {
	greet()
}
```

We will create a new file called `greet.go`:

```go
package main

import "fmt"

func greet() {
	fmt.Println("Hello from another file!")
}
```

Since both files have `package main`, they're part of the same package. No imports needed between them as they automatically see each other's functions. Now let's run the code:

```sh
# Option 1: Specify all files
go run main.go greet.go

# Option 2: Run all files in the directory (recommended)
go run .

# Option 3: Build an executable
go build
./hello-world
```

Use `go run .` or `go build` instead of listing individual files. It's less error-prone and works regardless of how many files you have.

### Multiple Packages

As projects grow, you'll want to organize code into multiple packages. Each package lives in its own subdirectory. They provide:

-   **Encapsulation**: Hide internal implementation details
-   **Namespace management**: Avoid naming conflicts
-   **Reusability**: Share code across different parts of your project
-   **Team collaboration**: Different teams can own different packages

It is best practice to name the directory the same as the package:

```text
.
├── go.mod
├── main.go
└── utils
    └── utils.go
```

As an example, let's create a `utils` package:

```go
// utils/utils.go
package utils

import "fmt"

// ExternalFunction is exported (capital first letter)
func ExternalFunction() {
	fmt.Println("Hello from utils package!")
}

// internalFunction is not exported (lowercase first letter)
func internalFunction() {
	fmt.Println("This is private to the utils package")
}
```

As shown above, in Go capitalization determines visibility:

-   **Uppercase first letter** = exported (public)
-   **Lowercase first letter** = unexported (private to the package)

To use our `utils` package, import it using the module path from `go.mod`:

```go
// main.go
package main

import (
	"hello-world/utils"
)

func main() {
	utils.ExternalFunction()
	// utils.internalFunction()  // Error: unexported
}
```

The import path is `module-name/directory-path`, not a URL. Go looks in your module for the package. If you have naming conflicts or want a shorter name, use an alias:

```go
package main

import (
	u "hello-world/utils"
)

func main() {
	u.ExternalFunction()
}
```

### Directory vs Package Names

As mentioned before, it's best practice to keep directory and package names the same. However, Go technically allows them to differ:

```go
// helpers/helper.go
package utils

import "fmt"

func ExternalFunction() {
	fmt.Println("Package utils in helpers directory")
}
```

When importing, use the **directory path**, but call functions using the **package name**:

```go
package main

import (
	"hello-world/helpers"
)

func main() {
	utils.ExternalFunction()
}
```

When you save the program though, the default linting will create an alias for more clarity like so:

```go
package main

import utils "hello-world/helpers"

func main() {
	utils.ExternalFunction()
}
```

This is confusing, so **avoid** it. Always name your directory the same as your package.

## Working with Third-Party Packages

Go makes it easy to use external packages. The Go module system automatically handles downloading and versioning. When you import a package that's not in your module, You first need to download it by running the following command:

```sh
go get github.com/Pallinder/go-randomdata
```

Now let's start importing into our program:

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

Now you can run the program and see the output in the terminal. By convention Go groups imports with a blank line between standard library and external packages:

```go
import (
	"fmt"

	"github.com/Pallinder/go-randomdata"
)
```

After adding a dependency, `go.mod` looks like:

```text
module hello-world

go 1.25.6

require github.com/Pallinder/go-randomdata v1.2.0 // indirect
```

**go.mod** lists your dependencies and their versions but **go.sum** contains cryptographic checksums of dependency content. This ensures:

-   You get the exact same code every time
-   No one can tamper with packages
-   Reproducible builds

Unlike Node.js's `node_modules`, Go doesn't store dependencies in your project. They're cached globally in `$GOPATH/pkg/mod` (usually `~/go/pkg/mod`) and shared across all your projects.

To download all the dependencies of a project, you can run the following command:

```sh
go mod download
```

For cleaning up unused dependencies, we have:

```sh
go mod tidy
```

### Vendoring

You can vendor dependencies (copy them into your project):

```sh
go mod vendor
```

This creates a `vendor/` directory which is useful for:

-   Ensuring dependencies are always available
-   Working in restricted environments
-   Faster CI builds

If we run the vendor command for the above program, our folder structure will look like this:

```text
.
├── go.mod
├── go.sum
├── main.go
└── vendor
    ├── github.com
    │   └── Pallinder
    │       └── go-randomdata
    │           ├── CHANGELOG.md
    │           ├── fullprofile.go
    │           ├── jsondata.go
    │           ├── LICENSE
    │           ├── postalcodes.go
    │           ├── random_data.go
    │           └── README.md
    └── modules.txt
```

When you run `go mod vendor`, Go copies all required dependencies into a local `vendor/` directory. However, Go does not automatically use the `vendor/` directory for builds in module mode. By default, the Go toolchain still resolves dependencies from the module cache (`$GOPATH/pkg/mod`). So in order to build with vendored dependencies, we have:

```sh
go build -mod=vendor
```

The `-mod=vendor` flag tells the Go toolchain explicitly, "Use the dependencies in the `vendor/` directory and do not resolve or download modules from anywhere else." In other words, you need to pass `-mod=vendor` to:

-   Force Go to use the vendored copies instead of the module cache
-   Prevent accidental network access during the build
-   Ensure the build is fully reproducible from the checked-in dependencies

Without `-mod=vendor`, the `vendor/` directory is essentially ignored during a normal module-aware build.

## Essential Go Tools

Go comes with a powerful set of built-in tools. Here are the ones you'll use most:

### Format Code

`go fmt` automatically formats your code according to Go's style guide:

```sh
go fmt ./...
```

Run this before every commit. Better yet, configure your editor to run it on save.

### Static Analysis

`go vet` examines code for common mistakes:

```sh
go vet ./...
```

It catches issues like:

-   Unreachable code
-   Invalid format strings in Printf
-   Misuse of sync.Mutex
-   And many more

Let's see that in action:

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Printf("Hello: %d\n", "World!")
}
```

If we run the above command, we will get `./main.go:8:21: fmt.Printf format %d has arg "World!" of wrong type string`.

### Clean Dependencies

Removes unused dependencies and adds missing ones:

```sh
go mod tidy
```

Run this regularly to keep `go.mod` clean.

## Package Organization Best Practices

For project structure guidelines, see [golang-standards/project-layout](https://github.com/golang-standards/project-layout).
