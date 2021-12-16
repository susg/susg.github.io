+++
categories = ["Low level design"]
date = "2021-12-16"
slug = "abstract-class-in-go"
title = "Abstract Class in Go"
tags = ["go", "design", "abstraction"]
[ author ]
  name = "Sushant Gupta"
+++

## What is an Abstract Class?

An __Abstract Class__ is the layout of the class. 
It defines several methods, some concrete and some abstract.
The child class extending the abstract class need to implement the abstract methods.
It can also override the implementation of a concrete method.

## Why use Abstract Class?

An abstract class is be used when we want to provide a common interface for different implementations.
But those multiple implementations have some methods in common which can be defined in the base 
class itself and used by all.

Consider the following usecase:
Let's say we are coding for a barista and need to write code for creating coffee and tea.
We can have a `Beverage` abstract class which can implement the common methods: `boilWater()` and `pourInCup()`.
Whereas the concrete classes: `Tea`  and `Coffee` can implement their own `prepareRecipe()` method.
![usecase](/img/go_abstract_class/usecase.drawio.svg)

## Why is it difficult in Go?

Though Go provides a custom type known as `interface`  which can be used to provide a common interface for different implementations.
But Go's `interface` doesn’t have fields and also it doesn’t allow the definition of methods inside it. Any type needs to implements all methods of interface to become of that interface type.

## Solution

We will make use of the [embedded field type](https://go.dev/ref/spec#Struct_types) in a struct.

Let's first define the `beverage` interface.
```Go
type beverage interface {
	boilWater()
	prepareRecipe()
	pourInCup()
}
```

Next we define a struct - `baseBeverage` which will implement the common methods of the `beverage` interface.
```Go
type baseBeverage struct {}

func (b baseBeverage) boilWater() {
	fmt.Println("Boiling water")
}

func (b baseBeverage) pourInCup() {
	fmt.Println("Pouring in beverage in cup")
}
```

Now we define our concrete beverages, i.e, `tea` and `coffee`.
These structs will contain `baseBeverage` as an embedded field and also implement the methods which is related  to them.

```Go
type tea struct {
	baseBeverage
}

func (t tea) prepareRecipe() {
	fmt.Println("Putting tea leaves")
}

type coffee struct {
	baseBeverage
}

func (c coffee) prepareRecipe() {
	fmt.Println("Putting coffee")
}
```

When struct A contains another struct B as an embedded field, the medthod sets of S include the promoted mehtods with receiver B.
Hence, when `tea` embeds `baseBeverage`, all the methods of `baseBeverage` can be used by objects of `tea` as well.

```Go
func main() {
	t := tea{}
	t.boilWater()
	t.prepareRecipe()
	t.pourInCup()
}
```

This allows us to use an object of struct `tea` as `beverage` interface type.
