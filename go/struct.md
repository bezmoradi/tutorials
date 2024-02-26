# Go > Struct

It is short for **struct**ure which can be thought of as a collection of properties that are related together but have different types. The struct data structure in Go is kind of the same as a class in other programming languages.

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	var p person
	fmt.Println(p) // {  }
}
```

As shown above, we have defined a struct called `person` then a variable of that type called `p`. When we don't initialize fields with real values, Go falls back to default values based on the field types; in other words, as the type of `firstName` and `lastName` is `string` and the default value for strings is an empty string, in the output we have `{  }`.

```go
func main() {
	var p person
	p.firstName = "John"
	p.lastName = "Doe"
	fmt.Println(p) // {John Doe}
}
```

As shown above, we have given some values to the struct fields. Just as with functions, we can consolidate fields of the same type within a struct:

```go
type person struct {
	firstName, lastName string
}
```

As another approach to create new variables of a specific type of struct we have:

```go
func main() {
	p := person{"John", "Doe"}
	fmt.Println(p) // {John Doe}
}
```

The main problem with the above way of creating variables off of a struct is that if down the road the **order** of keys inside the struct changes, we would face unordered values (This rule applies as long as the type of the fields that we have changed their orders is the same otherwise we will get compile-time error):

```go
type person struct {
	lastName  string
	firstName string
}

func main() {
	p := person{"John", "Doe"}
	fmt.Println(p.firstName) // Doe
	fmt.Println(p.lastName)  // John
}
```

So a safer way of creating a variable is as below because this way even if the order of struct fields changes, we won't get unexpected results:

```go
type person struct {
	lastName  string
	firstName string
}

func main() {
	p := person{firstName: "John", lastName: "Doe"}
	fmt.Println(p.firstName) // Doe
	fmt.Println(p.lastName)  // John
}
```

Another way of creating a variable is by using the `new` keyword:

```go
func main() {
	p := new(person)
	p.firstName = "John"
	p.lastName = "Doe"
	fmt.Println(p) // &{John Doe}
}
```

As we can see in the output, we have `&{John Doe}` and the `&` shows that creating a variable using the `new` keyword returns a pointer (More on this later). We can also create a struct anonymously when we only need one instance of such a struct:

```go
package main

import "fmt"

func main() {
	p1 := struct {
		firstName string
		lastName  string
	}{
		firstName: "John",
		lastName:  "Doe",
	}

	fmt.Println(p1)
}
```

We can also pass a struct to a function as follows:

```go
func printUserData(p person) {
	fmt.Println(p.firstName, p.lastName)
}

func main() {
	p1 := person{firstName: "John", lastName: "Doe"}
	printUserData(p1)
}
```

In the above example, a **copy** of the `person` will be sent to the `printUserData()` function. If we want to pass a reference (which in this case we should because there is not any reason for not doing that as we just need the original copy), we can refactor the above code as below:

```go
func printUserData(user *person) {
	fmt.Println((*user).firstName, (*user).lastName)
}

func main() {
	p1 := person{firstName: "John", lastName: "Doe"}
	printUserData(&p1)
}
```

The short-hand notation for the struct keys dereferencing is as:

```go
fmt.Println(user.firstName, user.lastName)
```

This way the compiler will automatically do the dereferencing process for us.

## Default Values of Struct Keys

Technically, the default value for keys in a struct without initial value is as follows:

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

The `Println` function does not show the fields; in order to show them, we need to use the `Printf` method as follows:

```go
func main() {
	var p person
	fmt.Printf("%+v", p) // {firstName: lastName:}
}
```

## Add Generic Types

In Go, we can add generics to functions; likewise, we can add generic types to structs as well:

```go
package main

import "fmt"

type person[T string | int] struct {
	lastName  string
	firstName string
	metadata  T
}

func main() {
	p := person[string]{
		firstName: "John",
		lastName:  "Doe",
		metadata:  "this is a generic type",
	}
	fmt.Println(p)
}
```

As shown above, type of the `metadata` can either be `string` or `int`.

## Nested Structs

As there is not any class concept in Go, we can compare this feature of structs to OOP inheritance. To define nested structs we have:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
	contact   contact
}

type contact struct {
	cell    string
	address string
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Doe",
		contact: contact{
			cell:    "1234567890",
			address: "L4X9A6",
		},
	}

	fmt.Println(p) // {John Doe {1234567890 L4X9A6}}
}
```

Another way of defining embedded structs is as follows:

```go
type person struct {
	firstName string
	lastName  string
	contact
}
```

Like JavaScript ES6 short-hand notation, we can remove the `contact` type if both the key and struct have the same name. To understand it better, let's see the following example:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

type adminUser struct {
	person      person
	permissions string
}

func main() {
	admin := adminUser{
		permissions: "READ/WRITE",
		person: person{
			firstName: "John",
			lastName:  "Doe",
		},
	}
	fmt.Println(admin) // {{John Doe} READ/WRITE}
}
```

Technically, the `adminUser` struct inherits all features of the `person` struct. Now let's add a method for the `person`:

```go
func (p person) printPersonData() {
	fmt.Println(p.firstName, p.lastName)
}
```

Now the `adminUser` also inherits this method and inside the `main()` function we can call it like this:

```go
admin.person.printPersonData()
```

It's too verbose if each time we type the full path of `admin.person`; to tackle that, we have to change the `adminUser` struct as follows:

```go
type adminUser struct {
	person
	permissions string
}
```

Instead of explicitly adding the nested type, we anonymously add that. From now on, we can easily call `admin.printPersonData()` (Still we can call it like `admin.person.printPersonData()` as well).

## Receiver Functions

We can also attach functions to structs and those functions can be called by variables of that type. Such functions are called Method:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func (p person) showUserData() {
	fmt.Println(p.firstName, p.lastName)
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p.showUserData()
}
```

Basically `(p person)` makes it possible for the `showUserData` function to be called on any variable of type `person` (In OOP world, it's like defining a method in a class). We can also mutate data using methods. First, let's see a non-working example then we'll fix it:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (p person) updateName(newName string) {
	p.firstName = newName
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p.updateName("Jane")
	p.showUserData() // {John Doe}
}
```

Although we are calling `p.updateName("Jane")` to update the name, `p.showUserData` shows the initial value! By default, Go works in a **Pass by Value** approach meaning the `updateName` function does not update the original struct but creates a copy with totally another location in memory. Here is the solution:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (p *person) updateName(newName string) {
	p.firstName = newName
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p.updateName("Jane")
	p.showUserData() // {Jane Doe}
}
```

Now let's break it down line by line:

-   `&person{}` assigns the memory address to `p`
-   `(p *person)` means we are working with the value that this memory address is referencing

The short-hand version is like so:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func (p person) showUserData() {
	fmt.Println(p)
}

func (p *person) updateName(newName string) {
	p.firstName = newName
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p.updateName("Jane")
	p.showUserData() // {Jane Doe}
}
```

In the above code we have removed the `&` while calling `person{}`. The code works fine because Go has a feature called "automatic referencing of methods" which simplifies working with pointers to objects; in other words, Go automatically takes the address of `p` when calling the method with a pointer receiver. When you define a method with a receiver that is a pointer to a type (`*person` in this case), Go allows you to call that method on both the pointer to the object and the object itself. In this example, when you call `p.updateName("Jane")`, Go automatically interprets this as if you had called `(&p).updateName("Jane")`, knowing that `updateName` has a receiver of type `*person`. This is a convenience provided by Go to make code cleaner and more readable. So, whether you explicitly use the pointer or not, Go internally handles the conversion between the object and its pointer, allowing both versions to work correctly.  
Now it's time to write a test:

```go
package main

import "testing"

func TestUpdateName(t *testing.T) {
	updatedName := "Jane"
	p := person{firstName: "John", lastName: "Doe"}
	p.updateName(updatedName)
	if p.firstName != updatedName {
		t.Errorf("expected %v but got %v", updatedName, p.firstName)
	}
}
```

## Constructor Functions

This is not a feature of Go but it's more of a design pattern:

```go
package main

import (
	"errors"
	"fmt"
)

type person struct {
	firstName string
	lastName  string
}

func newPerson(fname, lname string) (person, error) {
	if fname == "" || lname == "" {
		return person{}, errors.New("either first name of last name is empty")
	}
	return person{firstName: fname, lastName: lname}, nil
}

func main() {
	p, err := newPerson("John", "Doe")
	if err != nil {
		panic(err)
	}

	fmt.Println(p)
}
```

By convention, we can add `new` to the very beginning of the function name (Although it's not required). Now let's move this code into its own package called `user`:

```go
// ./user/user.go
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

We have created a new folder in the root directory called `user` and as we need to have access to the struct and function in this package, we have changed the name of `person` to `Person` and `newPerson` to `New` (We could also use `NewPerson`, but in Go it is conventional to name constructor functions as `New`). In order to use our newly-created package in the `main` package we have:

```go
package main

import (
	"fmt"

	"tutorials/user"
)

func main() {
	p, err := user.New("John", "Doe")
	if err != nil {
		panic(err)
	}
	fmt.Println(p) // {John Doe}
}
```

The path `"tutorials/user"` might be different for you depending on the module name inside the `go.mod` file. Now let's create a new user without using the constructor function; instead use the struct directly:

```go

func main() {
	user := user.Person{}
	fmt.Println(user)
}
```

Basically, as we start typing `user.Person{}`, the IDE does not give us the autocomplete because even if the name of the struct is PascalCase, all properties need to be that way as well. So to fix that we have:

```go
package user

import (
	"errors"
)

type Person struct {
	FirstName string
	LastName  string
}

func New(fname, lname string) (Person, error) {
	if fname == "" || lname == "" {
		return Person{}, errors.New("either first name of last name is empty")
	}

	return Person{FirstName: fname, LastName: lname}, nil
}
```

Then inside the `main()` function:

```go
func main() {
	user := user.Person{FirstName: "John", LastName: "Doe"}
	fmt.Println(user)
}
```

So this rule specifies whether the fields of a struct can be public to the outside world or not.
Now it's time to write some tests for our package:

```go
package user

import "testing"

func TestNewPerson(t *testing.T) {
	tests := []struct {
		name         string
		firstName    string
		lastName     string
		result       Person
		errorMessage string
	}{
		{
			name:         "valid input",
			firstName:    "John",
			lastName:     "Doe",
			result:       Person{FirstName: "John", LastName: "Doe"},
			errorMessage: "",
		},
		{
			name:         "empty FirstName",
			lastName:     "Doe",
			errorMessage: "either first name of last name is empty",
		},
		{
			name:         "empty LastName",
			firstName:    "John",
			errorMessage: "either first name of last name is empty",
		},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			p, err := New(test.firstName, test.lastName)
			if err != nil && err.Error() != test.errorMessage {
				t.Errorf("expected %v but got %v", test.errorMessage, err.Error())
			}
			if p != test.result {
				t.Errorf("expected %v but got %v", test.result, p)
			}
		})
	}
}
```

## Struct Tags

Struct tags are metadata that you add to your struct fields and that metadata can be picked up by any package or piece of code that cares about it:

```go
package main

import (
	"encoding/json"
	"fmt"
)

type person struct {
	firstName string `json:"first_name"`
	lastName  string `json:"last_name"`
}

func main() {
	p := person{firstName: "John", lastName: "Doe"}

	json, err := json.Marshal(p)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(json)) // {}

}
```

As shown above, the output is empty simply because all keys of the `person` struct must be **PascalCase** for the `json.Marshal` function to be able to parse them. Here is the fix:

```go
type person struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
}

func main() {
	p := person{FirstName: "John", LastName: "Doe"}
	json, err := json.Marshal(p)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(json)) // {"first_name":"John","last_name":"Doe"}
}
```

If we want to omit the struct key if empty, we can change it to

```go
type person struct {
	FirstName string `json:"first_name,omitempty"`
	LastName  string `json:"last_name,omitempty"`
}

func main() {
	p := person{FirstName: "John"}
	json, err := json.Marshal(p)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(json)) // {"first_name":"John"}
}
```

Keep in mind that before the `omitempty` keyword there should not be any white space.

## An Intro to Struct Validation

If we need to define a struct key as required, we can use a tag as follows:

```go
package main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type person struct {
	FirstName string `validate:"required"`
	LastName  string `validate:"required"`
}

func main() {
	p := person{}
	validate := validator.New()
	err := validate.Struct(p)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(p)
}
```

It outputs:

```text
Key: 'person.FirstName' Error:Field validation for 'FirstName' failed on the 'required' tag
Key: 'person.LastName' Error:Field validation for 'LastName' failed on the 'required' tag
```

And by adding required fields, we pass the validation step:

```go
func main() {
	p := person{FirstName: "John", LastName: "Doe"}
	validate := validator.New()
	err := validate.Struct(p)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(p) // {John Doe}
}
```

For the `validator` package to work properly, struct fields must be PascalCase.

## How to Embed A Struct in A Map

To do so we have:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

func main() {
	p1 := person{
		firstName: "John",
		lastName:  "Doe",
	}

	p2 := person{
		firstName: "Jane",
		lastName:  "Doe",
	}

	m := map[string]person{
		p1.firstName: p1,
		p2.firstName: p2,
	}

	fmt.Println(m) // map[Jane:{Jane Doe} John:{John Doe}]
}
```
