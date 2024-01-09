# Go > Array & Slice

-   In Go, we have two different types of data structures for storing a list of items:
    -   Array: Fixed length list of things
    -   Slice: An array that can grow or shrink
-   The key point here is that both Array and Slice elements **must** be of a unique type; meaning all members must be either `int`, `string`, or any other type

## Array

-   As it's fixed in size, it's mostly used for Go internals. The syntax to create an array in Go is as follows:

```go
package main

import "fmt"

func main() {
	numbers := [4]int{1, 2, 3, 4}
	fmt.Println(numbers)
}
```

-   `[]int` defines the type of the values which in this case is an array of integers and `[4]` defines the size of the array. We can also first initialize the array then add elements to it:

```go
package main

import "fmt"

func main() {
	var names [3]string
	fmt.Println(names)
}
```

-   If we see the output of the above program, `[ ]` will be printed and as the null value for strings in Go is an empty string, we see two empty spots in `[ ]`. To populate an existing array we can:

```go
names = [3]string{"Bez", "Far", "Dan"}
```

-   Or this way:

```go
names[0] = "Bez"
names[1] = "Far"
names[2] = "Dan"
```

-   To access an element of an array we can:

```go
fmt.Println(names[0])
```

-   To set a specific index of the array with a value we can:

```go
names[2] = "Dan"
```

## How to Get A Slice of An Array

-   To specify the start and end of the slice we can follow the `[inclusive:exclusive]` format

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	// [inclusive:exclusive]
	numbersSlice := numbers[2:7]

	fmt.Println(numbers)
	fmt.Println(numbersSlice)
}
```

-   In `numbers[2:7]` index 2 which holds number two is included and index 7 which holds number seven is NOT included; that's why we have `[2 3 4 5 6]`

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}


	// [:exclusive]
	numbersSlice := nums[:6]

	fmt.Println(numbers)
	fmt.Println(numbersSlice)
}
```

-   In the above program, `[:6]` means from the very beginning of the slice to the element at index 6 (It's not included) so we have `[0 1 2 3 4 5]`

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	// [inclusive:]
	numbersSlice := numbers[4:]

	fmt.Println(numbers)
	fmt.Println(numbersSlice)
}
```

-   In the above case, `[4:]` can be translated to `[inclusive:]` meaning from the element at index 4 which is number four including itself to the end or the slice which will be `[4 5 6 7 8 9]`

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	// [:]
	numbersSlice := numbers[:]
	// numbersSlice2 := numbers would yield the same result

	fmt.Println(numbers)
	fmt.Println(numbersSlice)
}
```

-   `[:]` means from the element at index 3 including itself to the end or the slice; so we get `[0 1 2 3 4 5 6 7 8 9]`

-   Technically speaking, a slice is a window to the original array; in other words, it does not copy anything from the original array but has a reference to part of that array that's why mutating a slice element would result in mutation in the original array as well. In other words, slice is pass by reference

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	numbersSlice := numbers
	numbersSlice[0] = 1000

	fmt.Println(numbers)
	fmt.Println(numbersSlice)
}
```

-   In the above example, we are mutating the second element of the slice at index 1 and if we print both the value of slice and also the original array we have:

```go
[1000 1 2 3 4 5 6 7 8 9]
[1000 1 2 3 4 5 6 7 8 9]
```

-   If in our specific use case we need to copy an array without referring to a pointer in memory we can:

```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	nums2 := make([]int, len(nums))
	copy(nums2, nums)
	nums2[0] = 1000
	fmt.Println(nums)
	fmt.Println(nums2)
}
```

-   In the output we have:

```text
[0 1 2 3 4 5 6 7 8 9]
[1000 1 2 3 4 5 6 7 8 9]
```

-   Another way of defining the length of an array is the `...` as follows:

```go
func main() {
	var names = [...]string{"Bez", "Far", "Dan"}

	fmt.Println(names)
}
```

-   `...` indicates that the size of the array should be determined by the **number of elements** provided in the **initialization**. Using `...` in this way can be helpful when you want to declare an array and have the compiler infer the size based on the provided elements, ensuring that the array is exactly large enough to hold those elements.

## How to Create Multidimensional Array In Go

-   To do that we can:

```go
func main() {
	numbers1 := []int{0, 1, 2, 3, 4, 5}
	numbers2 := []int{6, 7, 8, 9, 10}
	numbers3 := [][]int{numbers1, numbers2}

	fmt.Println(numbers3)
}
```

-   We can interpret `[][]int` as `[]` an array which has a type of `[]int` meaning an array of integers. We can optionally add `[2][]int{numbers1, numbers2}` meaning the length of our array is 2

### An Intro to `len()` and `cap()` Built-in Functions

-   The `len()` function shows the length of an array or slice

```go
numbers := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
fmt.Println(len(numbers))
```

-   It returns 10
-   The `cap()` function which is short for **Capacity** returns the capacity of the array or slice; in the following example, as the capacity of the array is 10, obviously it returns 10

```go
numbers := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
fmt.Println(cap(numbers))
```

-   If two arrays are of the same length, we can assign one to the other:

```go
func main() {
	numbers1 := []int{1, 2, 3}
	numbers2 := []int{10, 20, 30}
	numbers1 = numbers2
	fmt.Println(numbers1)
}
```

## Slice

-   To create a slice we have:

```go
names := []string{"Behzad", "Farnoosh", "Daniel"}
```

-   The only difference with creating an array is that we need to omit the hard-coded length
-   To add a new element to an slice we can:

```go
names := []string{"Behzad", "Farnoosh"}
updatedNames := append(names, "Daniel")
fmt.Println(names)
fmt.Println(updatedNames)
```

-   `append()` method does not modify the original array but creates a new one. In other words, it does not mutate the original slice; What Go does is that where there is a `append()` command, as slices are based on arrays which have fixed sizes, what Go would do is that it creates a new array with the new size and add the new element to that new array which our slice is based upon.
-   We can append multiple values in one go:

```go
updatedNames := append(names, "Daniel", "Alex")
```

-   As an alternative approach, instead of create a new variable, we can reassign the original array with the new value as follows:

```go
names := []string{"Behzad", "Farnoosh"}
names = append(names, "Daniel")
```

-   Keep in mind that instead of the `:=` operator we have used the `=` which is used for assigning a value

## How to Remove An Element from An Slice

-   There isn't any build-in function like `append()` to removing element and the workaround for this scenario is as follows:

```go
names := []string{"Behzad", "Farnoosh", "Daniel"}
names = names[1:]
fmt.Println(names)
```

-   We have actually removed the first element from the slice. To remove the last element we have:

```go
names = names[:len(names)-1]
```

-   In the following example, we want to remove number 5 from the slice

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	numbers = append(numbers[:5], numbers[6:]...)
	fmt.Println(numbers)
}
```

## Pass by Reference

-   In Go, slices are what you call "reference types." When you pass a slice to a function, you're actually passing a reference to the underlying array that holds the slice's elements, not a copy of the entire slice:

```go
package main

import "fmt"

func main() {
	names := []string{"Bez", "Far", "Dan"}

	update(names)
	fmt.Println(names)
}

func update(s []string) {
	s[0] = "Behzad"
}
```

-   So, when you pass the `names` slice to the `update` function, you're giving it the memory address where the slice's data (which includes the array of strings) is stored. This means that any changes made to the slice inside the `update` function directly affect the original underlying array.

## How to Create A Slice of Structs

-   To do that we have:

```go
package main

import "fmt"

type Product struct {
	id    int
	title string
}

func main() {
	products := []Product{
		Product{id: 1, title: "First"},
		Product{id: 2, title: "Second"},
	}

	fmt.Println(products)
}
```

-   The IDE would warn that the `Product` keyword in an element like `Product{id: 1, title: "First"}` is redundant just because Go would figure out that the underlying type of that element is `Product`. So we can simplify the above code as follows:

```go
func main() {
	products := []Product{
		{id: 1, title: "First"},
		{id: 2, title: "Second"},
	}

	fmt.Println(products)
}
```

-   We can also first initialized the variable then add elements to it:

```go
func main() {
	var products []Product

	products = append(products, Product{id: 1, title: "first"})

	fmt.Println(products)
}
```

## How to Use The Spread Operator

-   The `append()` function can also be used for merging two arrays/slices:

```go
package main

import "fmt"

func main() {
	names1 := []string{"Behzad"}
	names2 := []string{"Farnoosh", "Daniel"}

	names1 = append(names1, names2...)
	fmt.Println(names1)
}
```

-   The key point here is that if we need to pass a array/slice as the second argument to the `append()` function, we have to use the special syntax `...` which spreads the elements like so:

```go
names1 = append(names1, "Farnoosh", "Daniel")
```

## Use The `make()` function to Create Slices

-   The `make` function initializes a slice with a specified length and capacity. It's useful when you know the initial size of the slice you need, and optionally, the capacity to which it might grow without having to dynamically resize as elements are appended:

```go
slice := make([]int, 5, 10)
```

-   This creates a slice with length 5 and capacity 10. In the output we have:

```text
[0 0 0 0 0]
```

-   Now if change the above program to:

```go
func main() {
	slice := make([]int, 0, 10)

	fmt.Println(slice)
}
```

-   In the output we would see:

```text
[]
```

-   Basically, the main difference lies in the fact the we when added 5 as the second argument to the `make()` function, we asked that function to initialize the slice and as the default value for `int` type is 0, we have `[0 0 0 0 0]`. In the second case, when we passed 0 as the second argument, it means that we do not want to initialize the slice that's why we get `[]`
-   But the slice literal `[]int{}` initializes an empty slice. This method is suitable when the size or capacity of the slice isn't predefined, and you plan to append elements dynamically based on your program's logic:

```go
slice := []int{}
```

-   This creates an empty slice
-   Technically, you can use `make()` when you know the initial size and optionally the capacity of the slice, especially in scenarios where you want to allocate contiguous memory for the slice elements upfront or when dealing with larger slices to avoid **frequent reallocations** (Because every time we add a new element, behind the scenes a new array is created)

```go
func main() {
	names := make([]string, 2, 10)
	names = append(names, "Dan")
	fmt.Println(names)
}
```

-   Using `make()` function as above is more memory-efficient because each time we add an element, Go does not have to allocate new space because already we have enough space for ten elements and only it will start adding new space when we go beyond 10 elements.
-   In the output we have:

```text
[  Dan]
```

-   As shown above, the first two slots are empty (indexes 0 and 1) and the `append()` function starts adding after those empty slots which in this case is index 2. In order to fill in the first two slots we have:

```go
func main() {
	names := make([]string, 2, 10)
	names[0] = "Bez"
	names[1] = "Far"
	names = append(names, "Dan")

	fmt.Println(names)
}
```

-   Now in the output we have:

```text
[Bez Far Dan]
```

-   I we use the slice literal and want to add an element to it as by index as follows:

```go
func main() {
	names := []string{}
	names[0] = "Bez"

	fmt.Println(names)
}
```

-   We would get error because there is no index of 0 in our slice. To add elements in this case, we need to use the `append()` function
-   Something good to know about the `make()` function is that in the following example `make([]int, 0, 10)` created a slice with the capacity of 10 and when we add numbers 11 and 12 to our slice, Go compiler automatically doubles the slice capacity from 10 to 20. By the same token, when we add numbers 13 to 21, again the compiler doubles the capacity from 20 to 40

```go
func main() {
	nums := make([]int, 0, 10)
	nums = append(nums, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
	fmt.Println(nums, len(nums), cap(nums))
	fmt.Println("----------")
	nums = append(nums, 11, 12)
	fmt.Println(nums, len(nums), cap(nums))
	fmt.Println("----------")
	nums = append(nums, 13, 14, 15, 16, 17, 18, 19, 20, 21)
	fmt.Println(nums, len(nums), cap(nums))
}
```

-   In the output we have:

```text
[1 2 3 4 5 6 7 8 9 10] 10 10
----------
[1 2 3 4 5 6 7 8 9 10 11 12] 12 20
----------
[1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21] 21 40
```

## Index out of Range Error

-   In programming languages like JS/TS, if we reference an index which does not exist, we would get `undefined` value but in Go we would get compile-time error:

```go
package main

import "fmt"

func main() {
	var names = []string{"Bez", "Far", "Dan"}

	fmt.Println(names[3])
}
```

-   In the output we have:

```text
go run .
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
main.main()
	/Users/behzadmoradi/Documents/projects/playgrounds/go/main.go:8 +0x24
exit status 2
```

-   In the above example, if we work with array, VSCode warns us but for slices we won't get any warning inside the editor but when we run the program, in either case we would get compile-time error
