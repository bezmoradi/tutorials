# Go > Strings

-   In order to remove decimals we can:

```go
userInput := 100.01
formattedString := fmt.Sprintf("%.0f", userInput)
fmt.Println(formattedString)
```

-   `%.0f` means zero decimal; if we want to have one decimal place we can do `%.1f` which in the output we would get `100.0`

## Add Line Break

-   To add line breaks we can:

```go
fmt.Print("Start")
fmt.Print("\n")
fmt.Print("The end")
```

## Get The Type of A Variable

-   To get the type we can use the `Printf()` method as follows:

```go
var userInput float64 = 100.01
fmt.Printf("The type of user input is: %T", userInput)
```

## Strings in Multiple Lines

-   As in JS, we can use backticks to print strings in multiple lines

```go
fmt.Print(`This is the first line
	this is the second line
	this is the third line`)
```

## How to Replace A Specific Character 
- `strings` is a built-in package in Go which makes working with string super easy. As an example, if you want to replace something in string you can:
```go
cleanOutput := strings.ReplaceAll("This is to test", " ", "_")
```