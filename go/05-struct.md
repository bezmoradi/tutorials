# Go > Struct

-   It is short for structure which can be thought of a collection of properties that are related together but with different types
-   The Struct data structure in Go is kind of the same as `Object` in JavaScript

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{firstName: "Bez", lastName: "Mor"}
	p2 := person{"Far", "Fahim"}
	var p3 person

	fmt.Println(p1, p2, p3)
}
```

-   As shown above, we have defined a Struct called `person` then created three different variables in three different ways of using Structs (If we need to define the `person` struct in another file within the same package, we have to rename it to `Person` to be able to export it from that file).

```go
p2 := person{"Far", "Fahim"}
```

-   The main problem with the above way of creating variables off of a Struct is that if down the road the **order** of keys inside the Struct changes, we would faced unordered values
-   We can also create a struct anonymously

```go
package main

import "fmt"

func main() {
	p1 := struct {
		firstName string
		lastName  string
	}{
		firstName: "Bez",
		lastName:  "Mor",
	}

	fmt.Println(p1)
}
```

-   We can also pass a struct to a function as follows:

```go
func main() {
	p1 := person{firstName: "Bez", lastName: "Mor"}

	printUserData(p1)
}

func printUserData(user person) {
	fmt.Println(user.firstName, user.lastName)
}
```

-   In the above example, a copy of the `user` will be sent to the `printUserData()` function. If we want to pass a reference (which in this case we should because there is not any reason for not doing that as we just need the original copy), we can refactor the above code as below:

```go
func main() {
	p1 := person{firstName: "Bez", lastName: "Mor"}

	printUserData(&p1)
}

func printUserData(user *person) {
	fmt.Println((*user).firstName, (*user).lastName)
	// The short-hand notation for the struct keys dereferencing is as:
	// fmt.Println(user.firstName, user.lastName)
}
```

## Default Values of Struct Keys

-   Technically, the default value for keys in a Struct without initial value is as follows:

```text
+-----------+-----------+
|  string   |    ""     |
+-----------+-----------+
|    int    |     0     |
+-----------+-----------+
|   float   |     0     |
+-----------+-----------+
|   bool    |   false   |
+-----------+-----------+
```

-   The `fmt.Println()` method does not show the default values; in order to show them, we need to use the `fmt.Printf()` method as follows:

```go
func main() {
	var p3 person

	fmt.Printf("%+v", p3)
}
```

-   And in the output we will see:

```text
{firstName: lastName:}
```

-   In order to assign values to a struct we can:

```go
func main() {
	p1 := person{firstName: "Bez", lastName: "Mor"}
	p2 := person{"Far", "Fahim"}
	var p3 person

	p3.firstName = "Dan"
	p3.lastName = "Mor"

	fmt.Println(p1, p2, p3)
}
```

## Nested Structs

-   To define nested structs we have:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
	contact   contactInfo
}

type contactInfo struct {
	cell    string
	address string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
		contact: contactInfo{
			cell:    "4372444407",
			address: "L3Y3B9",
		},
	}

	fmt.Println(p1)
}
```

-   Another way of defining embedded structs is as follows:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
	contactInfo
}

type contactInfo struct {
	cell    string
	address string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
		contactInfo: contactInfo{
			cell:    "4372444407",
			address: "L3Y3B9",
		},
	}

	fmt.Println(p1)
}
```

-   As seen in

```go
type person struct {
	firstName string
	lastName  string
	contactInfo
}
```

-   Like JS short-hand notation, we can remove the key if both the key and Struct have the same name
-   As there is not any class concept in Go, we can properly compare this feature of structs to OOP Inheritance. To make it more clear, let's see the following example:

```go
type Person struct {
	FirstName string
	LastName  string
}

type AdminUser struct {
	SuperPassword string
	Person Person
}
```

-   Technically, the `AdminUser` struct inherits all features of the `Person` struct. In order to use it we have:

```go
admin := user.AdminUser{SuperPassword: "1234567", Person: user.Person{FirstName: "Bez", LastName: "Moradi"}}
```

-   Now let's add a method for the `Person`:

```go
func (p Person) PrintPersonData() {
	fmt.Println(p.FirstName, p.LastName)
}
```

-   It only prints the first name and last name. Now the `AdminUser` also inherits this method and inside the `main()` function we can call it like this:

```go
admin.Person.PrintPersonData()
```

-   It's too verbose to each time type `admin.Person`; to tackle that, we have to change the `AdminUser` struct as follows:

```go
type AdminUser struct {
	SuperPassword string
	Person
}
```

-   Meaning instead of explicitly added the nested type, we anonymously add that. From now on, we can easily call `admin.PrintPersonData()`

## Receiver Functions And Structs

-   We can also attach methods to structs and those methods can be called by variables with that struct type:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
	}

	p1.showUserData()
}

// (p person) is called the Receiver
func (p person) showUserData() {
	fmt.Println(p)
}
```

-   Basically what `(p person)` instructs is that `showUserData()` function can be called on any variable of type `person` (In OOP world, it's like defining a method in a class)
-   We can also mutate data using struct methods:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
	}

	p1.updateName("Behzad")
	p1.showUserData()
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (p person) updateName(newName string) {
	p.firstName = newName
}
```

-   If we run the above program we would get:

```text
{Bez Mor}
```

-   Although we are calling `p1.updateName("Behzad")` to update the name, `p1.showUserData()` shows the initial value. By default, Go language works in a **Pass by Value** approach meaning the `updateName()` function does not update the original struct but created a new one with the new value. Here is the solution:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
	}

	p1Pointer := &p1
	p1Pointer.updateName("Behzad")
	p1.showUserData()
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (pointerToPerson *person) updateName(newName string) {
	(*pointerToPerson).firstName = newName
}
```

-   Now let's break it down line by line:

    -   `p1Pointer := &p1` assigns the memory address of `p1` to `p1Pointer`
    -   `func (pointerToPerson *person)` means that we are working with a pointer to a `person` type
    -   `(*pointerToPerson)` means we are working with the value that this memory address is referencing

-   The short-hand version is like so:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Mor",
	}

	p1.updateName("Behzad")
	p1.showUserData()
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (p *person) updateName(newName string) {
	p.firstName = newName
}
```

-   The code works fine because Go has a feature called "automatic referencing of methods" which simplifies working with pointers to objects. When you define a method with a receiver that is a pointer to a type (`*person` in this case), Go allows you to call that method on both the pointer to the object and the object itself. In this example, when you call `p1.updateName("Behzad")`, Go automatically interprets this as if you had called `(&p1).updateName("Behzad")`, knowing that `updateName` has a receiver of type `*person`. This is a convenience provided by Go to make code cleaner and more readable. So, whether you explicitly use the pointer or not `(p1Pointer.updateName("Behzad"))` vs. `p1.updateName("Behzad")`, Go internally handles the conversion between the object and its pointer, allowing both versions to work correctly.

## Constructor Functions

-   This is not a feature of Go but it's more of a design pattern:

```go

func main() {
	p1, err := newPerson("Bez", "Moradi")

	if err != nil {
		panic(err)
	}

	fmt.Println(p1)
}

func newPerson(fname, lname string) (person, error) {
	if fname == "" || lname == "" {
		return person{}, errors.New("either first name of last name is empty")
	}

	return person{firstName: fname, lastName: lname}, nil
}
```

-   By convention, we can add `new` to the very beginning of the function name (Although it's not required)
-   Now let's move this code into its own package called `user`:

```go
package user

import (
	"errors"
)

type Person struct {
	firstName string
	lastName  string
}

func New(fname, lname string) (Person, error) {
	if fname == "" || lname == "" {
		return Person{}, errors.New("either first name of last name is empty")
	}

	return Person{firstName: fname, lastName: lname}, nil
}
```

-   We have created a new folder in the root directory called `user` an as we need to have access to the struct and function in this package, we have change the name of `person` to `Person` and `newPerson` to `New` (We could also use `NewPerson`, but in Go it is conventional to name constructor functions as `New`)
-   Now in order to use our newly-created package in the `main` package we have:

```go
package main

import (
	"fmt"

	"github.com/bez/go-practice/user"
)

func main() {
	p1, err := user.New("Bez", "Moradi")

	if err != nil {
		panic(err)
	}

	fmt.Println(p1)
}
```

-   Now let's create a new user without using the constructor function; instead use the struct directly:

```go

func main() {
	user := user.Person{}
	fmt.Println(user)
}
```

-   Basically, as we start typing `user.Person{}`, the IDE does not give us the autocomplete because even if the name of the struct is PascalCase, all properties need to be that way as well. So to fit that we have:

```go
package user

import (
	"errors"
)

type Person struct {
	FirstName string
	FastName  string
}

func New(fname, lname string) (Person, error) {
	if fname == "" || lname == "" {
		return Person{}, errors.New("either first name of last name is empty")
	}

	return Person{FirstName: fname, FastName: lname}, nil
}
```

-   Then inside the `main()` function:

```go
func main() {
	user := user.Person{FirstName: "Bez", LastName: "Moradi"}
	fmt.Println(user)
}
```

-   So this rule specifies whether the fields of a struct can be public to the outside world or not

## Struct Tags

-   Struct tags are metadata that you add to your struct fields and that metadata can be picked up by any package or piece of code that cares about it

```go
type Note struct {
	Title    string    `json:"title"`
	Content  string    `json:"content"`
	CreateAt time.Time `json:"createdAt"`
}

func (note Note) Save() error {
	json, err := json.Marshal(note)

	if err != nil {
		return err
	}

	return os.WriteFile("notes.json", json, 0644)
}
```
- The **important** thing about the above code is that all keys of the `Note` struct must be PascalCase for the `json.Marshal` function to be able to parse them.
-   If we want to omit the struct key if empty, we can change it to

```go
CreateAt time.Time `json:"createdAt,omitempty"`
```

-   Actually `json.Marshal()` is a function that if finds any struct tags, will start using it and would create such an output

```text
{"title":"learn go","content":"this is to test","createdAt":"2023-12-21T18:32:05.40338-05:00"}
```

## An Intro to `binding` tag

-   If we need to define a struct key as required, we can use a tag as follows:

```go
type Note struct {
	Title    string    `json:"title" binding:"required"`
	Content  string    `json:"content"`
	CreateAt time.Time `json:"createdAt"`
}
```

## How to Embed A Struct in A Map

-   To do so we have:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
	age       int
}

func main() {
	p1 := person{
		firstName: "Bez",
		lastName:  "Moradi",
		age:       39,
	}

	p2 := person{
		firstName: "Far",
		lastName:  "Fahim",
		age:       36,
	}

	m := map[string]person{
		p1.lastName: p1,
		p2.lastName: p2,
	}

	for _, v := range m {
		fmt.Println(v.age)
	}
}
```
