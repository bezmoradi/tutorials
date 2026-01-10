# Go > Introduction to The `log` Package

One of the differences between the `fmt.Println` function and `log.Println` is that the latter also prints the date and time:

```go
package main

import (
	"errors"
	"log"
)

func main() {
	err := errors.New("this is a custom-made error")
	log.Print(err) // 2024/01/08 18:40:38 this is a custom-made error
}
```

The `fmt` package formats an error value by calling its `Error` method. One other difference is that with the `log` package we can define **where** the log needs to go:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	f, err := os.Create("logs.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer f.Close()
	log.SetOutput(f)
	_, err = os.Open("NON_EXISTING_FILE")
	if err != nil {
		log.Println(err)
	}
}
```

In this program, first we create a brand-new file called `logs.txt` then by calling `log.SetOutput(f)`, we instruct the `log` package to send logs to that file instead of `Stdout`.
