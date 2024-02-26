# Go > Interface

In Go, interfaces serve as blueprints that describe the expected behaviors of types. Much like the saying "If it walks like a duck and it quacks like a duck, then it must be a duck", if a type adheres to the methods specified by an interface, it is treated as if it implements that interface (For more information, refer to [Duck typing](https://en.wikipedia.org/wiki/Duck_typing)).  
Technically, we don't need to manually link a struct with an interface because Go handles the matching process automatically. If the type exhibits the required behaviors, it's effectively a 'duck' in the context of that interface and Go seamlessly recognizes and utilizes it.

```go
package main

import "fmt"

type payable interface {
	pay() string
}

type stripe struct {
	name string
}

func (s stripe) pay() string {
	return "Paid by " + s.name
}

type paypal struct {
	name string
}

func (p paypal) pay() string {
	return "Paid by " + p.name
}

func main() {
	stripe := stripe{name: "Stripe"}
	paypal := paypal{name: "PayPal"}
	payments := []payable{stripe, paypal}

	for _, payment := range payments {
		fmt.Println(payment.pay())
	}
}
```

In the above program we have a `payable` interface and as both `stripe` and `paypal` structs implement that interface by exposing the `pay` method, they can be considered as a `payable` type; that's why they can easily be added to the `payments` variable which is a slice of `paylable` type. We can refactor the above code as follows:

```go
func doPayment(p payable) string {
	return p.pay()
}

func main() {
	stripe := stripe{name: "Stripe"}
	paypal := paypal{name: "PayPal"}
	payments := []payable{stripe, paypal}
	for _, payment := range payments {
		fmt.Println(doPayment(payment))
	}
}
```

In the above program, the `doPayment(p payable)` function accepts any params that matches that interface. Basically, there is no **direct** link between `stripe` and `paypal` structs and the `payable` interface; Go takes care of it when we call `doPayment()` with each of them. If they follow the rules of the `payable` interface by having a `pay` method, we are good to go otherwise we'll get an error at compile time.  
Now let's add some tests:

```go
package main

import "testing"

func TestPayable(t *testing.T) {
	assertPayment := func(t *testing.T, payable payable, want string) {
		t.Helper()
		got := payable.pay()
		if got != want {
			t.Errorf("expected %v but got %v", want, got)
		}
	}

	t.Run("test stripe struct", func(t *testing.T) {
		stripe := stripe{name: "Stripe"}
		assertPayment(t, stripe, "Paid by Stripe")
	})

	t.Run("test paypal struct", func(t *testing.T) {
		paypal := paypal{name: "PayPal"}
		assertPayment(t, paypal, "Paid by PayPal")
	})
}
```

We can refactor the above test as below also:

```go
package main

import "testing"

func TestPayable(t *testing.T) {
	assertPayment := func(t *testing.T, payable payable, want string) {
		t.Helper()
		got := payable.pay()
		if got != want {
			t.Errorf("expected %v but got %v", want, got)
		}
	}

	tests := []struct {
		name    string
		payable payable
		want    string
	}{
		{
			name:    "Stripe",
			payable: stripe{name: "Stripe"},
			want:    "Paid by Stripe",
		},
		{
			name:    "PayPal",
			payable: paypal{name: "PayPal"},
			want:    "Paid by PayPal",
		},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			assertPayment(t, test.payable, test.want)
		})
	}
}
```

## Type Assertion

Type assertion is a way to access the underlying concrete value of an interface:

```go
func (s stripe) stripeSpecificMethod() string {
	return s.name + " specific method"
}

func (s paypal) paypalSpecificMethod() string {
	return s.name + " specific method"
}
```

First, let's add the above two methods each for one of the types we have already defined. Now let's update the `doPayment` function as follows:

```go
func doPayment(p payable) {
	if isStripe, ok := p.(stripe); ok {
		fmt.Println(isStripe.stripeSpecificMethod())
	}
	if isPaypal, ok := p.(paypal); ok {
		fmt.Println(isPaypal.paypalSpecificMethod())
	}
}
```

`p.(stripe)` is the type assertion syntax which returns two values: the concrete value which in this case is `stripe` type which is stored inside the `isStripe` variable and also a boolean value stored in `ok` which will be truthy if the assertion is successful. We can improve the above code using the `switch` statement:

```go
func doPayment(p payable) {
	switch t := p.(type) {
	case stripe:
		fmt.Println(t.stripeSpecificMethod())
	case paypal:
		fmt.Println(t.paypalSpecificMethod())
	}
}
```

If we need to define an interface which includes a method with input params, we can either make named params or just specify the type:

```go
package main

import (
	"fmt"
	"strconv"
)

type payable interface {
	pay(amount float64) string
}

type stripe struct {
	name string
}

func (s stripe) pay(amount float64) string {
	return strconv.Itoa(int(amount)) + " paid by " + s.name
}

type paypal struct {
	name string
}

func (p paypal) pay(amount float64) string {
	return strconv.Itoa(int(amount)) + " paid by " + p.name
}

func doPayment(p payable, amount float64) {
	fmt.Println(p.pay(amount))
}

func main() {
	stripe := stripe{name: "Stripe"}
	doPayment(stripe, 50.50)

	paypal := paypal{name: "PayPal"}
	doPayment(paypal, 100.10)
}
```

## Embedded Interfaces

The `payable` interface tells Go that any value of type `payable` must have all the methods of the `payable` and also the `showBalance` method:

```go
package main

import (
	"fmt"
	"strconv"
)

type balance interface {
	showBalance() float64
}

type payable interface {
	balance
	pay(amount float64) string
}

type stripe struct {
	name    string
	deposit float64
}

func (s *stripe) pay(amount float64) string {
	s.deposit = s.deposit - amount

	return strconv.Itoa(int(amount)) + " paid by " + s.name
}

func (s stripe) showBalance() float64 {
	return s.deposit
}

type paypal struct {
	name    string
	deposit float64
}

func (p *paypal) pay(amount float64) string {
	p.deposit = p.deposit - amount

	return strconv.Itoa(int(amount)) + " paid by " + p.name
}

func (p paypal) showBalance() float64 {
	return p.deposit
}

func doPayment(p payable, amount float64) {
	fmt.Println(p.pay(amount))
	fmt.Println("Balance is:", p.showBalance())
}

func main() {
	stripe := stripe{name: "Stripe", deposit: 100}
	doPayment(&stripe, 50.50)

	paypal := paypal{name: "PayPal", deposit: 300}
	doPayment(&paypal, 100.10)
}
```

## How to Explicitly Assign A Concrete Type to An Abstract Type

In the following program we have an interface called `payable` which can be considered as an **Abstract type** and also a struct called `stripe` which is considered as a **Concrete Type**. By assigning a concrete type to an abstract one, we ask Go to check if the concrete variable **has** all properties of the interface; in other words, as long as the `stripe` struct has the `pay` method, the compiler is OK with that:

```go
package main

import "fmt"

type payable interface {
	pay() string
}

type stripe struct {
	name string
}

func (s stripe) pay() string {
	return "Paid by " + s.name
}

func main() {
	var payer payable
	stripe := stripe{name: "Stripe"}
	payer = stripe
	fmt.Println(payer.pay())
}
```

## Let's mimic How Interface in Go Can Be Useful while Dealing With Different DBs

In the following example, the `DB` interface helps us add a layer of abstraction to work with different databases:

```go
package main

import "fmt"

type DB interface {
	save(userModel)
	retrieve(int) userModel
}

type userModel struct {
	id        int
	firstName string
	lastName  string
}

type mongodb struct {
	name string
	data map[int]userModel
}

func (m mongodb) save(u userModel) {
	m.data[u.id] = u
}

func (m mongodb) retrieve(id int) userModel {
	return m.data[id]
}

type postgres struct {
	name string
	data map[int]userModel
}

func (m postgres) save(u userModel) {
	m.data[u.id] = u
}

func (m postgres) retrieve(id int) userModel {
	return m.data[id]
}

func setter(db DB, u userModel) {
	db.save(u)
}

func getter(db DB, id int) userModel {
	return db.retrieve(id)
}

func main() {
	u1 := userModel{id: 1, firstName: "John", lastName: "Doe"}
	u2 := userModel{id: 2, firstName: "Jane", lastName: "Doe"}
	u3 := userModel{id: 3, firstName: "John", lastName: "Doe"}

	mongodb := mongodb{name: "MongoDB", data: map[int]userModel{}}
	postgres := postgres{name: "Postgres", data: map[int]userModel{}}

	setter(mongodb, u1)
	setter(mongodb, u2)
	setter(postgres, u3)

	fmt.Println(getter(postgres, 3))
}
```

## How to User Interfaces with Generics?

One of the use cases of interfaces in Go is with generics. To know more about that, you can visit the **Generics** tutorial in this series.
