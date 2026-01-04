# Go > Godoc

Godoc parses Go source code - including comments - and produces documentation as HTML or plain text. The end result is documentation tightly coupled with the code it documents (To read more, visit [Godoc: documenting Go code](https://go.dev/blog/godoc)). To install Godoc, run the following command in the terminal:

```text
$ go install golang.org/x/tools/cmd/godoc@latest
```

To see that in action, let's first create a simple project. The important thing about the module name is that it has to start with a domain name. For example:

```text
module github.com/bezmoradi/go-tuts

go 1.22.0
```

As the next step, let's create a folder called `helpers` then inside it create a file called `helpers.go`:

```go
// Package summary
//
// The main description of the package
// on multiple lines
package helpers

// Constant
const Something = "this is a text"

// Add adds two integers
func Add(a, b int) int {
	return a + b
}
```

`// Package summary` is the summary of the package which usually is a short description. If we need to have a longer description as our main description of the package, we have to add `//` on a new line then add our main description.  
An important note on Godoc is that there should not be an empty line between the comment and what we are commenting on; in other words, the following snippet would not produce any Godoc simply because there is an empty line between the statements and comments:

```go
// Package summary
//
// The main description of the package
// on multiple lines

package helpers
```

In this helper file, we have a constant and a function. The key point about these two is that their names start with a uppercase letter that's why they are visible to the outside world + they will be added to the Godoc. To test, if we change `Something` to `something`, this constant will not be included in the documentation. Now it's time to generate the documentation by running the following command inside terminal:

```text
$ godoc -http :1234
```

Basically, `:1234` refers to any open port on localhost. For example, if `1234` is open, you can see the documentation by entering `http://localhost:1234` inside any browser.

## Examples

Alongside test suites, we can also add examples to `_test.go` files. The good thing about examples is that they are added to the project documentation to show other devs how a piece of code is used plus they are executed like normal tests so you can be confident they reflect what the code actually does (Unlike examples that can be found outside the codebase such as a readme file which often become out of date and incorrect compared to the actual code because they don't get checked).  
Unlike normal test functions, though, example functions take no arguments and begin with the word `Example` instead of `Test`. To do that, we need to create a new file called `helpers_test.go` including the following logic:

```go
package helpers

import "fmt"

func ExampleAdd() {
	sum := Add(1, 2)
	fmt.Println(sum)
	// Output: 3
}
```

By going to `http://localhost:1234` and refreshing the page, we'll see that the above example is added beneath the `Add` function definition. The good news about examples is that they are run as tests meaning if the logic is updated but the example is not, our test suite fails and this feature guarantees that our examples are always up-to-date.  
In the above example, `// Output: 3` is an important piece because if we omit it, this example function won't be run a part of our test suite. Examples without output comments are useful for demonstrating code that cannot run as unit tests, such as that which accesses the network, while guaranteeing the example at least compiles.  
Multiple examples can be provided for a given identifier by using a suffix beginning with an underscore followed by a lowercase letter. Each of these examples documents the `Add` function:

```go
package helpers

import "fmt"

func ExampleAdd() {
	sum := Add(1, 2)
	fmt.Println(sum)
	// Output: 3
}

func ExampleAdd_second() {
	sum := Add(2, 2)
	fmt.Println(sum)
	// Output: 4
}

func ExampleAdd_third() {
	sum := Add(2, 3)
	fmt.Println(sum)
	// Output: 5
}
```

In the docs they are shown as:

```text
Example
Example (Second)
Example (Third)
```

Sometimes we need more than just a function to write a good example. In situations like this, we need to add only one example function inside our test file:

```go
package helpers

import "fmt"

const (
	one = 1
	two = 2
)

func Example() {
	sum := Add(one, two)
	fmt.Println(sum)
	// Output: 3
}
```

In this case, the above example will be shown as a package-level example inside the docs.

## How to See The Documentation Online?

If your repository is public, the docs can be accessed on `https://pkg.go.dev/<repo>`. For example, the repository of this tutorial is accessible on `https://pkg.go.dev/github.com/bezmoradi/go-tuts`.
