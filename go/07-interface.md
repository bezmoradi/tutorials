# Go > Interface

-   In Go, we as developers do not have to put together any manual code to link a struct with an interface; Go takes care of all the matching for us!
-   In the following program, we have an interface called `paymentInterface` and the `doPayment(p paymentInterface)` function accepts any params that matches this interface. Basically, there is no **direct** link between `stripe` and `paypal` structs and the `paymentInterface` interface. Go takes care of it when we call `doPayment()` with each of them. If they follow the rules of the `paymentInterface` interface, we are good to go otherwise at compile time we get an error

```go
package main

import "fmt"

type paymentInterface interface {
	pay() string
}

type stripe struct {
	name string
}

type paypal struct {
	name string
}

func main() {
	stripe := stripe{name: "Stripe"}
	paypal := paypal{name: "PayPal"}

	doPayment(stripe)
	doPayment(paypal)
}

func doPayment(p paymentInterface) {
	fmt.Println(p.pay())
}

func (s stripe) pay() string {
	return "Paid by" + s.name
}

func (p paypal) pay() string {
	return "Paid by" + p.name
}
```

-   As another example of interface we have:

```go
package main

import "fmt"

type Shape interface {
	getArea() float64
}

type triangle struct {
	height float64
	base   float64
}

type square struct {
	sideLength float64
}

func main() {
	t := triangle{base: 10, height: 10}
	s := square{sideLength: 10}

	printArea(t)
	printArea(s)
}

func printArea(shape Shape) {
	fmt.Println(shape.getArea())
}

func (s square) getArea() float64 {
	return s.sideLength * s.sideLength
}

func (t triangle) getArea() float64 {
	return 0.5 * t.base * t.height
}
```

## Interface Contracts with Input Params

-   If we need to define an interface which includes a method with input params, we just need to define the type and contrary to a language like TS, there is no need to add a param name

```go
type Printable interface {
	print(string, string) string
}
```

## Naming Interface

-   It's not forced but it's convention that if your interface has only one method, the name of the interface is the same as that method + "er"

```go
type saver interface {
	save()
}
```

## Embedded Interfaces

-   The `embeddedInterface` tells Go that any value of type `embeddedInterface` must have all the methods of the `saver` and also the `display()` method

```go
type saver interface {
	save()
}

type embeddedInterface interface {
	saver
	display()
}
```

## How to Explicitly Assign A Concrete Type to An Abstract Type

-   In the following program we have an interface called `subscribeable` which can be considered as an **Abstract type** and also a struct called `internetSubscription` which is considered as a **Concrete Type**. By assigning a concrete type to an abstract one, we ask Go to check if the concrete variable **HAS** all properties of the interface; in other words, as long as the `internetSubscription` type has the `subsribe()` method on it, the compiler is OK with that.

```go
package main

import "fmt"

func main() {
	is := internetSubscription{packageName: "5G", packageAmout: 100.00}

	var internetSubscriber subscribeable
	internetSubscriber = is

	internetSubscriber.subscribe()
}

// This is a concrete type
type internetSubscription struct {
	packageName  string
	packageAmout float64
}

func (is internetSubscription) subscribe() {
	fmt.Println("subscribed!")
}

// This is an abstract type
type subscribeable interface {
	subscribe()
}
```

## Let's mimic How Interface in Go Can Be Useful while Dealing With Different DBs

-   In the following example, the `accessor` interface helps us add a layer of abstraction to work with different databases:

```go
package main

import "fmt"

type person struct {
	firstName string
	lastName  string
}

type mongo map[int]person

func (m mongo) save(n int, p person) {
	m[n] = p
}

func (m mongo) retrieve(n int) person {
	return m[n]
}

type postgres map[int]person

func (p postgres) save(n int, per person) {
	p[n] = per
}

func (p postgres) retrieve(n int) person {
	return p[n]
}

type accessor interface {
	save(int, person)
	retrieve(int) person
}

func set(a accessor, n int, p person) {
	a.save(n, p)
}

func get(a accessor, n int) person {
	return a.retrieve(n)
}

func main() {
	p1 := person{firstName: "Bez", lastName: "Moradi"}
	p2 := person{firstName: "Far", lastName: "Fahim"}
	p3 := person{firstName: "Dan", lastName: "Moradi"}

	myMongo := mongo{}
	myPostgres := postgres{}

	set(myMongo, 1, p1)
	set(myMongo, 2, p2)
	set(myPostgres, 1, p3)

	fmt.Println(get(myPostgres, 1))
}
```
