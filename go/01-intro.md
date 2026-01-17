# Go > Introduction

For decades, single-threaded programs benefited from steady increases in CPU clock speeds. Based on trends before 2003, industry projections anticipated 10 GHz processors by 2005. However, fundamental physical constraints prevented this trajectory from continuing! Around 2004-2005, chip manufacturers encountered what became known as the "power wall". Increasing clock frequency requires higher voltage, and power consumption scales cubically with voltage. This relationship produces heat that exceeds practical cooling capabilities. As an example, Intel cancelled its planned 10 GHz "Tejas" processor in 2004, and clock speeds plateaued around 4 GHz.

The industry responded by increasing **core count** rather than clock speed. AMD introduced multi-core processors in 2004, followed by Intel in 2005. In March 2005, Herb Sutter published an influential article titled "The Free Lunch Is Over", arguing that developers could no longer rely on hardware improvements for performance gains and would need to write concurrent code to utilize multiple cores. However, concurrent programming with existing languages remained complex and error-prone. In September 2007, Robert Griesemer, Rob Pike, and Ken Thompson began designing a new language at Google with the explicit goal of making concurrent programming simpler and safer. That language became [Go](https://go.dev/).

## Installing Go on macOS

Before we start coding, let's install Go on macOS using Homebrew:

```bash
brew install go
```

Verify the installation by running:

```bash
go version
```

You should see output like: `go version go1.24.0 darwin/amd64` (version may vary). To update Go to the latest version:

```bash
brew upgrade go
```

## Your First Go Program: Hello World

Let's write your first Go program. Create a new directory for your project:

```bash
mkdir hello-world
cd hello-world
```

Create a file named `main.go` with the following content:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

Run it:

```bash
go run main.go
```

You should see: `Hello, World!`

**What just happened?** Go compiled your code to a temporary executable and ran it. For development, this is convenient. For production, you'll want to build a standalone executable.

## Understanding Packages and Modules

### What Are Packages?

A **package** is a collection of Go source files in the same directory that work together. Packages are Go's way of organizing and reusing code.

**Why packages exist**: Without packages, all code would be in one giant file, making it impossible to organize large projects or share code between projects. Packages solve this by providing namespaces and encapsulation.

### The Special `package main`

In Go, `package main` has a special meaning—it tells Go that this package should be compiled into an executable program, not a library.

**Key rules**:

-   Executable programs must have exactly one `package main`
-   The `main` package must contain exactly one `main()` function
-   The `main()` function is the entry point where program execution begins
-   You cannot have multiple `main()` functions in your program

**Example**:

```go
package main

import "fmt"

func main() {
	fmt.Println("This is the entry point")
}
```

**Note**: Packages like `fmt` don't have a `main()` function because they're libraries meant to be used by other programs, not run directly.

### What Are Modules?

Before Go 1.11, Go used `$GOPATH` for managing dependencies. Since Go 1.11, Go uses **modules**—a collection of related packages with versioning support.

**Why modules exist**: Modules solve dependency management, version control, and reproducible builds. They let you specify exactly which version of each dependency your project uses.

If we try to build an executable by running `go build` without a module, we would get an error:

```text
$ go build
go: go.mod file not found in current directory or any parent directory; see 'go help modules'
```

This error tells us we need to create a module. Let's do that:

```bash
$ go mod init hello-world
go: creating new go.mod: module hello-world
go: to add module requirements and sums:
	go mod tidy
```

**Module naming**: For local projects, you can use simple names like `hello-world`. For projects you plan to publish, use a path like `github.com/username/project`.

Now a file called `go.mod` is created in your directory:

```text
module hello-world

go 1.24
```

**Understanding `go.mod`**:

-   **First line** (`module hello-world`): Your module's name/path
-   **Second line** (`go 1.24`): Minimum Go version required
-   **Additional lines** (when you add dependencies): List of dependencies with their versions

Now if we run `go build`, the program compiles successfully:

```bash
$ go build
```

This creates an executable file named `hello-world` (or `hello-world.exe` on Windows). Run it:

```bash
$ ./hello-world          # Linux/macOS
$ .\hello-world.exe      # Windows
Hello, World!
```

**Key point**: The `go build` command requires `package main` to create an executable. Without it, Go doesn't know where the program should start.

## Organizing Code: Multiple Files in One Package

As your code grows, you'll want to split it across multiple files for better organization. All files in the same directory with the same `package` declaration belong to that package.

**Why this matters**: You can have hundreds of files in a single package, making it easy to organize related functionality without complex import statements.

**Example**: Let's split our code into two files:

```go
// main.go
package main

func main() {
	greet()
}
```

```go
// greet.go
package main

import "fmt"

func greet() {
	fmt.Println("Hello from another file!")
}
```

Since both files have `package main`, they're part of the same package. No imports needed between them—they automatically see each other's functions.

**Running the code**:

```bash
# Option 1: Specify all files
$ go run main.go greet.go

# Option 2: Run all files in the directory (recommended)
$ go run .

# Option 3: Build an executable
$ go build
$ ./hello-world
```

**Best practice**: Use `go run .` or `go build` instead of listing individual files. It's less error-prone and works regardless of how many files you have.

## Organizing Code: Multiple Packages

As projects grow, you'll want to organize code into multiple packages. Each package lives in its own subdirectory.

**Why multiple packages**: They provide:

-   **Encapsulation**: Hide internal implementation details
-   **Namespace management**: Avoid naming conflicts
-   **Reusability**: Share code across different parts of your project
-   **Team collaboration**: Different teams can own different packages

### Package Structure

**Best practice**: Name the directory the same as the package:

```text
.
├── go.mod
├── main.go
└── utils
    └── utils.go
```

**Example**: Create a `utils` package:

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

**Important**: In Go, capitalization determines visibility:

-   **Uppercase first letter** = exported (public)
-   **Lowercase first letter** = unexported (private to the package)

### Importing Local Packages

To use our `utils` package, import it using the module path from `go.mod`:

```go
// main.go
package main

import (
	"hello-world/utils"  // module-name/package-path
)

func main() {
	utils.ExternalFunction()  // Works!
	// utils.internalFunction()  // Error: unexported
}
```

**Key insight**: The import path is `module-name/directory-path`, not a URL. Go looks in your module for the package.

### Import Aliases

If you have naming conflicts or want a shorter name, use an alias:

```go
package main

import (
	u "hello-world/utils"  // Alias 'u' for 'utils'
)

func main() {
	u.ExternalFunction()  // Use the alias
}
```

### Directory vs Package Names

**Best practice**: Keep directory and package names the same. However, Go technically allows them to differ:

```go
// helpers/some-file.go
package utils  // Package name is 'utils'

import "fmt"

func ExternalFunction() {
	fmt.Println("Package utils in helpers directory")
}
```

When importing, use the **directory path**, but call functions using the **package name**:

```go
// main.go
package main

import (
	"hello-world/helpers"  // Import by directory path
)

func main() {
	utils.ExternalFunction()  // Use package name
}
```

**This is confusing, so avoid it.** Always name your directory the same as your package.

## Working with Third-Party Packages

Go makes it easy to use external packages. The Go module system automatically handles downloading and versioning.

### Adding Dependencies

When you import a package that's not in your module, Go will automatically download it when you build or run:

```go
// main.go
package main

import (
	"fmt"
	"github.com/Pallinder/go-randomdata"
)

func main() {
	fmt.Println(randomdata.City())
}
```

Run your code:

```bash
$ go run .
go: downloading github.com/Pallinder/go-randomdata v1.2.0
London  # Or another random city
```

Go automatically:

1. Downloaded the package
2. Added it to `go.mod`
3. Added checksums to `go.sum`
4. Compiled and ran your code

### Understanding go.mod and go.sum

After adding a dependency, `go.mod` looks like:

```text
module hello-world

go 1.24

require github.com/Pallinder/go-randomdata v1.2.0 // indirect
```

**go.mod** lists your dependencies and their versions.

**go.sum** contains cryptographic checksums of dependency content. This ensures:

-   You get the exact same code every time
-   No one can tamper with packages
-   Reproducible builds

**Note**: Unlike Node.js's `node_modules`, Go doesn't store dependencies in your project. They're cached globally in `$GOPATH/pkg/mod` (usually `~/go/pkg/mod`) and shared across all your projects.

### Module Commands

**Adding/updating a dependency**:

```bash
$ go get github.com/Pallinder/go-randomdata@latest  # Latest version
$ go get github.com/Pallinder/go-randomdata@v1.2.3  # Specific version
$ go get github.com/Pallinder/go-randomdata@v1      # Latest v1.x.x
```

**Downloading all dependencies** (for teammates):

```bash
$ go mod download
```

**Cleaning up unused dependencies**:

```bash
$ go mod tidy
```

**Important change (Go 1.17+)**: `go get` is primarily for adding/updating dependencies in `go.mod`. To install executable tools, use `go install`:

```bash
# Wrong (deprecated)
$ go get github.com/some/tool

# Correct
$ go install github.com/some/tool@latest
```

### Import Organization

Go convention: group imports with a blank line between standard library and external packages:

```go
import (
	"fmt"        // Standard library
	"os"

	"github.com/Pallinder/go-randomdata"  // External packages
)
```

Tools like `goimports` do this automatically.

## Essential Go Tools

Go comes with a powerful set of built-in tools. Here are the ones you'll use most:

### go fmt - Format Code

`go fmt` automatically formats your code according to Go's style guide:

```bash
$ go fmt ./...  # Format all files in current directory and subdirectories
```

**Best practice**: Run this before every commit. Better yet, configure your editor to run it on save.

### go vet - Static Analysis

`go vet` examines code for common mistakes:

```bash
$ go vet ./...
```

It catches issues like:

-   Unreachable code
-   Invalid format strings in Printf
-   Misuse of sync.Mutex
-   And many more

### go mod tidy - Clean Dependencies

Removes unused dependencies and adds missing ones:

```bash
$ go mod tidy
```

Run this regularly to keep `go.mod` clean.

### go build - Compile

Compile your program:

```bash
$ go build                    # Build for current platform
$ go build -o myapp          # Specify output name
$ GOOS=linux go build        # Cross-compile for Linux
$ GOOS=windows go build      # Cross-compile for Windows
```

### goimports - Advanced Formatting

`goimports` does everything `go fmt` does, plus manages imports automatically:

```bash
$ go install golang.org/x/tools/cmd/goimports@latest
$ goimports -w .
```

**Highly recommended**: Configure your editor to run `goimports` on save instead of `go fmt`.

## Advanced Module Concepts

### Semantic Versioning

Go modules use semantic versioning (semver):

```text
v1.2.3
│ │ └─ Patch: Bug fixes (backward compatible)
│ └─── Minor: New features (backward compatible)
└───── Major: Breaking changes (NOT backward compatible)
```

**Key rule**: Major versions v2+ require a suffix in the module path:

```go
module github.com/user/project/v2  // Major version 2

import "github.com/user/project/v2/pkg"
```

This allows different major versions to coexist in the same project.

### The replace Directive

The `replace` directive lets you use a local copy of a dependency:

```go
// go.mod
module hello-world

go 1.24

require github.com/someone/package v1.2.3

replace github.com/someone/package => ../local-copy
```

**Use cases**:

-   Testing local changes before publishing
-   Using a fork temporarily
-   Working on multiple modules simultaneously

### Workspaces (Go 1.18+)

Workspaces let you work on multiple modules simultaneously:

```bash
$ go work init ./module1 ./module2
$ go work use ./module3
```

Creates a `go.work` file:

```text
go 1.24

use (
    ./module1
    ./module2
    ./module3
)
```

**When to use**: When developing multiple related modules locally (e.g., a main application and a library it uses).

**Important**: Don't commit `go.work` to version control—it's for local development only.

### Private Modules

To use private repositories (GitHub, GitLab, Bitbucket):

```bash
$ export GOPRIVATE=github.com/yourcompany/*
```

Add to `~/.bashrc` or `~/.zshrc` for persistence.

For authentication:

```bash
$ git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"
```

### Module Proxies

By default, Go uses the public module proxy at `https://proxy.golang.org`. This provides:

-   Fast downloads worldwide
-   Availability even if the original source disappears
-   Checksum database for security

To disable (for private modules):

```bash
$ export GOPRIVATE=github.com/yourcompany/*
```

### Vendoring

You can vendor dependencies (copy them into your project):

```bash
$ go mod vendor
```

This creates a `vendor/` directory. Useful for:

-   Ensuring dependencies are always available
-   Working in restricted environments
-   Faster CI builds

Build with vendored dependencies:

```bash
$ go build -mod=vendor
```

## Package Organization Best Practices

For project structure guidelines, see [golang-standards/project-layout](https://github.com/golang-standards/project-layout).
