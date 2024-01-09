# Go > Custom Types

-   We create our own custom types off of built-in Go types:

```go
package main

import "fmt"

type customString string

func main() {
	var userName customString = "behzad"
	userName.logString()
}

func (s customString) logString() {
	fmt.Println(s)
}
```

-   In the above program, we have created a custom type called `customString` which extends the `string` type.
-   We can also add function to our own type
