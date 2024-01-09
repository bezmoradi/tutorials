# Go > Read & Write to Files

-   In order to write to a file we have:

```go
package main

import (
	"os"
)

func main() {
	os.WriteFile("sample.txt", []byte("In the name of the most high"), 0644)
}
```

-   Technically, `0644` is high enough to give us the permissions to read and write to the newly-created file
-   `[]byte` is called byte slice and converts a string to something like:

```text
[73 110 32 116 104 101 32 110 97 109 101 32 111 102 32 116 104 101 32 109 111 115 116 32 104 105 103 104]
```

-   In order to read from file we have:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	data, err := os.ReadFile("sample.txt")

	if err != nil {
		fmt.Println("Error", err)
		panic(err)
	}

	fmt.Println(string(data))
}
```

-   As `ReadFile` method returns a byte slice, we need to convert it back to string that's why we did `string(data)`
-   In order to kill the app, instead of `panic()` we can also use `os.Exit(1)`

## Error Handling in Go

-   Contrary to other programming languages, if a built-in function throws an error, the application does not crash.
-   For example, if `os.ReadFile("sample.txt")` does not find the file, it returns an empty byte slice

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func main() {
	result, possibleErrors := readFileContents("sample.txt")
	fmt.Println(result)
	fmt.Println(possibleErrors)
}

func readFileContents(fileName string) (string, error) {
	byteSliceData, err := os.ReadFile(fileName)
	stringData := string(byteSliceData)

	if err != nil {
		fmt.Println("Error in readFileContents()", err)

		return stringData, errors.New("sorry could not fild the file")
	}

	return stringData, nil
}
```

-   In the above code which the refactored version of the previous app, `readFileContents()` function return both the result set and any possible error. `return stringData, nil` state that if there is no error, instead of error return `nil` which is an equivalent of `null` in TS/JS

## Save Data as JSON

-   In order to turn a struct into a JSON then save it in `.json` file, we have:

```go
func (note Note) Save() error {
	json, err := json.Marshal(note)

	if err != nil {
		return err
	}

	return os.WriteFile("notes.json", json, 0644)
}
```

-   As `WriteFile` might throw an error, we can return it to match the return type of this function

## Reading from A File Line by Line

-   In order to read the contents of a file line by line we have:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("items.txt")

	if err != nil {
		fmt.Println("Failed to open the file")

		return
	}

	scanner := bufio.NewScanner(file)

	var lines []string

	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}

	fmt.Println(lines)

	file.Close()
}
```

-   When we open a file but forget to close it, this situation can lead to a few potential issues like:
    -   Resource leakage because leaving files open consumes system resources
    -   Some systems might keep the file locked while it's open. If another process or part of your program tries to access the same file, it might be denied access or encounter unexpected behavior.
    -   If your program writes to the file and doesn't close it properly, the written data might not be completely flushed or saved to the file. This can lead to incomplete or corrupted data if the program terminates unexpectedly
-   For these reasons, we have to make sure to close the file when we are done that's why in the above program at the end we are calling `file.Close()`
-   There is also another way to close a file using the `defer` keyword

```go

func main() {
	file, err := os.Open("items.txt")

	if err != nil {
		fmt.Println("Failed to open the file")

		return
	}

	defer file.Close()

	scanner := bufio.NewScanner(file)

	var lines []string

	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}

	fmt.Println(lines)
}
```

-   Still we are calling the `file.Close()` but this time added the `defer` keyword before it. We when execute a function with the `defer` keyword before it, Go will not execute the function right away but it invokes it when the surrounding function (in this case `main()`) finished; either by throwing an error or completing successfully. Long story short, Go will call that function at the right time
-   Another benefit of the `defer` keyword is that if we need to manually call the `file.Close()` in multiple places, by using `defer file.Close()` we just need to call it once and Go will take care of the rest
