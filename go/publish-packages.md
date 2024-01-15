# Go > How to Publish Open Source Package

-   In order to publish a package, first we new to initialize a Go module:
-   The important note while create a third-party package is that the Go module name while running `go mod init <name>` MUST match the GitHub repo otherwise other apps won't be able to fetch our package

```text
$ go mod init github.com/behzadmoradi/<repository_name>
```

-   Next, we have to create a file with whatever name we want:

```go
// main.go

package ructur

import (
	"fmt"
)

func SayHi() {
	fmt.Println("Hi")
}
```

-   As we want our functions be visible to the outside world, they have to start with a Capital letter as in `SayHi()`
-   In this example, the name of the package is identical to the GitHub repository name which is `ructur`
-   As the next step, we push all changes to the remote repository
-   Also we need to create a Git tag and push it. To do so, after pushing our changes, we run `git tag "v1.0.0"` then run `git push --tags`
-   Now in order to use our package, first we need to create a new project the run:

```text
$ go get github.com/behzadmoradi/ructur@v1.0.0
```

-   The use it like so

```go
package main

import "github.com/behzadmoradi/ructur"

func main() {
	ructur.SayHi()
}
```

-   Now let's imagine we need to create another package in our open source project

```go
// ./utils/utils.go
package utils

func AddUp(a, b int) int {
	return a + b
}
```

-   After pushing the changes to the remote repository and creating a new tag for our commit, in order to use it, first we need to update our package in the other project:

```text
$ go get github.com/behzadmoradi/ructur@v1.0.20
```

-   Then use it like so:

```go
package main

import (
	"fmt"

	"github.com/behzadmoradi/ructur"
	"github.com/behzadmoradi/ructur/utils"
)

func main() {
	ructur.SayHi()
	fmt.Println(utils.AddUp(1, 10))
}
```

## Versioning Tips

-   When creating a new version of your package, as long as it's minor version (Based on semantic versioning, the number in the middle or the one all the way to right changes), there won't be any issues BUT in order to create a major version (like `v2.0.0`), it's important to follow certain conventions to ensure proper module resolution
-   To handle version `v2` and higher, there are two ways:

### Subdirectories

-   One approach is to create a folder called `v2` and place all of files/folders inside it and make sure the change the `go.mod` file as follows:

```text
module github.com/behzadmoradi/ructur/v2

go 1.21.5
```

-   Then commit the changes and create a tag like `v2.0.0` and push it.
-   When we want to use the `v2` in other apps, we have to run

```text
$ go get github.com/behzadmoradi/ructur/v2@v2.0.0
```

-   Still we can get back to `v1` and make some changes and push it. Other apps which need to get the latest version of `v1` can do

```text
$ go get github.com/behzadmoradi/ructur@v1.100.100
```

### Branches

-   To apply the second version via Git branches, let's create a new branch called `v3` the update the `go.mod` file as follows:

```text
module github.com/behzadmoradi/ructur/v3

go 1.21.5
```

-   Then push the new branch to GitHub then create a tag for it:

```text
$ git tag v3.0.0
$ git push --tags
```

-   Then we can download `v3` as follows in other apps:

```text
$ go get github.com/behzadmoradi/ructur/v3@v3.0.0
```
