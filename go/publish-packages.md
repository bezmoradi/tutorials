# Go > How to Publish Open Source Package

In order to publish a package, first we new to initialize a Go module. The important note while create a third-party package is that the Go module name while running `go mod init <name>` MUST match the GitHub repo otherwise other apps won't be able to download our package:

```text
$ go mod init github.com/<username>/something
```

Next, we have to create a file with whatever name we want:

```go
// main.go
package something

import (
	"fmt"
)

func SayHi() {
	fmt.Println("Hi")
}
```

As we want our functions be visible to the outside world, they have to start with an uppercase first letter as in `SayHi` function. In this example, the name of the package is identical to the GitHub repository name which is `something`. As the next step, we push all changes to the remote repository. Also we need to create a Git tag and push it. To do so, after pushing our changes, we run `git tag "v1.0.0"` then run `git push --tags`. Now in order to use our package, first we need to create a new project then run:

```text
$ go get github.com/<username>/something@v1.0.0
```

Afterwards, import and use it this way:

```go
package main

import "github.com/<username>/something"

func main() {
	something.SayHi()
}
```

Now let's imagine we need to create another package in our open source project:

```go
// ./utils/utils.go
package utils

func AddUp(a, b int) int {
	return a + b
}
```

After pushing the changes to the remote repository and creating a new tag for our commit, in order to use it, first we need to update our package in the other project:

```text
$ go get github.com/<username>/something@v1.0.2
```

Then start using that as follows:

```go
package main

import (
	"fmt"

	"github.com/<username>/something"
	"github.com/<username>/something/utils"
)

func main() {
	ructur.SayHi()
	fmt.Println(utils.AddUp(1, 10))
}
```

## Versioning Tips

When creating a new version of your package, as long as it's either a backward compatible feature or a bug fix (Based on semantic versioning, the number in the middle or the one all the way to right), there won't be any issues BUT in order to create a major version (like `v2.0.0`), it's important to follow certain conventions to ensure proper module resolution. To handle version `v2` and higher, there are two ways:

### Subdirectories

One approach is to create a folder called `v2` and place all of files/folders inside it and make sure to change the `go.mod` file as follows:

```text
module github.com/<username>/something/v2

go 1.21.5
```

Then commit the changes and create a tag like `v2.0.0` and push it. When we want to use the `v2` in other apps, we have to run:

```text
$ go get github.com/<username>/something/v2@v2.0.0
```

Still we can get back to `v1` and make some changes and push it. Other apps which need to get the latest version of `v1` can do:

```text
$ go get github.com/<username>/something@v1.100.100
```

### Branches

To apply the second version via Git branches, let's create a new branch called `v3` the update the `go.mod` file as follows:

```text
module github.com/<username>/something/v3

go 1.21.5
```

Then push the new branch to GitHub then create a tag for it:

```text
$ git tag v3.0.0
$ git push --tags
```

Now we can download `v3` as follows in other apps:

```text
$ go get github.com/<username>/something/v3@v3.0.0
```

## An Intro to Go Workspaces

Let's imagine we need to create a package on our local machine and before pushing it on a remote repository like GitHub, we need to import it in another project and test it. This is where workspaces come to play. To create a Go workspace, let's first create a directory called `my-workspace` then inside that directory run:

```text
$ go work init
```

We'll see that a new file called `go.work` is created inside that directory. As the next step, let's create two stand-alone projects inside that directory; one called `lib` and the other one called `app`:

```text
.
├── app
│   ├── go.mod
│   └── main.go
├── go.work
└── lib
    ├── go.mod
    └── utils
        └── utils.go
```

After running `go mod init github.com/bezmoradi/lib` and `go mod init github.com/bezmoradi/app` inside the `lib` and `app` folders respectively, now we need to add both folders to the workspace we already created by running `go work use app` and `go work use lib`. If we take a look at the `go.work` file, we will see:

```text
go 1.21.5

use (
	./app
	./lib
)
```

From now on, we are able to use the `lib` project inside the `app` project. Let's add a new package inside the `lib` project called `utils/utils.go`:

```go
package utils

import "fmt"

func Greet(name string) {
	fmt.Printf("Hi %v", name)
}
```

And the `main.go` file inside the `app` folder will look like this:

```go
package main

import "github.com/bezmoradi/lib/utils"

func main() {
	utils.Greet("John")
}
```

By running `go run main.go` inside the `app` folder, we can successfully use the package we created.

## An Alternative to Using Workspaces

There is also another approach we can use to import another project while in development environment. To to that, let's first delete the `go.work` file then change the `go.mod` file inside the `app` folder as follows:

```text
module github.com/bezmoradi/app

go 1.21.5

replace github.com/bezmoradi/lib => ../lib

require github.com/bezmoradi/lib v0.0.0-00010101000000-000000000000 // indirect
```
By using the `replace` keyword, we are asking the Go compiler to replace the real path of `github.com/bezmoradi/lib` with `../lib` meaning going one directory up then use whatever found inside the `lib` folder. Finally, we have to run `go get github.com/bezmoradi/lib` which results in adding the last line. From now on, we can simply use the `lib` project inside the `app`. 