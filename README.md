# 🍕 `github.com/elliotchance/pie`
[![GoDoc](https://godoc.org/github.com/elliotchance/pie?status.svg)](https://godoc.org/github.com/elliotchance/pie)
[![Build Status](https://travis-ci.org/elliotchance/pie.svg?branch=master)](https://travis-ci.org/elliotchance/pie)
[![codecov](https://codecov.io/gh/elliotchance/pie/branch/master/graph/badge.svg)](https://codecov.io/gh/elliotchance/pie)

**Enjoy a slice!** `pie` is a code generator for dealing with slices that
focuses on type safety, performance and immutability.

- [Quick Start](#quick-start)
  * [Install/Update](#install-update)
  * [Built-in Types](#built-in-types)
  * [Custom Types](#custom-types)
  * [Limiting Functions Generated](#limiting-functions-generated)
- [Functions](#functions)
- [FAQ](#faq)
  * [What are the requirements?](#what-are-the-requirements-)
  * [What are the goals of `pie`?](#what-are-the-goals-of--pie--)
  * [How do I contribute a function?](#how-do-i-contribute-a-function-)
  * [Why is the emoji a slice of pizza instead of a pie?](#why-is-the-emoji-a-slice-of-pizza-instead-of-a-pie-)

# Quick Start

## Install/Update

```bash
go get -u github.com/elliotchance/pie
```

## Built-in Types

`pie` ships with some slice types ready to go (pun intended). These include:

- `type`[`Strings`](https://godoc.org/github.com/elliotchance/pie/pie#Strings)`[]string`
- `type`[`Float64s`](https://godoc.org/github.com/elliotchance/pie/pie#Float64s)`[]float64`
- `type`[`Ints`](https://godoc.org/github.com/elliotchance/pie/pie#Ints)`[]int`

These can be used without needing `go generate`. For example:

```go
package main

import (
    "fmt"
    "strings"

    "github.com/elliotchance/pie/pie"
)

func main() {
    name := pie.Strings{"Bob", "Sally", "John", "Jane"}.
        Unselect(func (name string) bool {
            return strings.HasPrefix(name, "J")
        }).
        Transform(strings.ToUpper).
        Last()

    fmt.Println(name) // "SALLY"
}
```

## Custom Types

Annotate the slice type in your source code:

```go
type Car struct {
    Name, Color string
}

//go:generate pie Cars.*
type Cars []Car
```

Run `go generate`. This will create a file called `cars_pie.go`. You should
commit this with the rest of your code. Run `go generate` any time you need to
add more types.

Now you can use the slices:

```go
cars := Cars{
    {"Bob", "blue"},
    {"Sally", "green"},
    {"John", "red"},
    {"Jane", "red"},
}

redCars := cars.Select(func(car Car) bool {
    return car.Color == "red"
})

// redCars = Cars{{"John", "red"}, {"Jane", "red"}}
```

Or, more complex operations can be chained:

```go
cars.Unselect(func (car Car) {
        return strings.HasPrefix(car.Name, "J")
    }).
    Transform(func (car Car) Car {
        car.Name = strings.ToUpper(car.Name)

        return car
    }).
    Last()

// Car{"SALLY", "green"}
```

## Limiting Functions Generated

The `.*` can be used to generate all functions. This is easy to get going but
creates a lot of unused code. You can limit the functions generated by chaining
the function names with a dot syntax, like:

```go
//go:generate myInts.Average.Sum myStrings.Select
```

This will only generate `myInts.Average`, `myInts.Sum` and `myStrings.Select`.

# Functions

| Function     | String | Number | Struct| Maps | Big-O    | Description |
| ------------ | :----: | :----: | :----:| :--: | :------: | ----------- |
| `All`        | ✓      | ✓      | ✓     |      | n        | All will return true if all callbacks return true. If the list is empty then true is always returned. |
| `Any`        | ✓      | ✓      | ✓     |      | n        | Any will return true if any callbacks return true. If the list is empty then false is always returned. |
| `Append`     | ✓      | ✓      | ✓     |      | n        | A new slice with the elements appended to the end. |
| `AreSorted`  | ✓      | ✓      |       |      | n        | Check if the slice is already sorted. |
| `AreUnique`  | ✓      | ✓      |       |      | n        | Check if the slice contains only unique elements. |
| `Average`    |        | ✓      |       |      | n        | The average (mean) value, or a zeroed value. |
| `Bottom`     | ✓      | ✓      | ✓     |      | n        | Gets n elements from bottom. |
| `Contains`   | ✓      | ✓      | ✓     |      | n        | Check if the value exists in the slice. |
| `Extend`     | ✓      | ✓      | ✓     |      | n        | A new slice with the elements from each slice appended to the end. |
| `Each`       | ✓      | ✓      | ✓     |      | n        | Perform an action on each element. |
| `First`      | ✓      | ✓      | ✓     |      | 1        | The first element, or a zeroed value. |
| `FirstOr`    | ✓      | ✓      | ✓     |      | 1        | The first element, or a default value. |
| `Join`       | ✓      |        |       |      | n        | A string from joining each of the elements. |
| `JSONString` | ✓      | ✓      | ✓     |      | n        | The JSON encoded string. |
| `Keys`       |        |        |       | ✓    | n        | Returns all keys in the map (in random order). |
| `Last`       | ✓      | ✓      | ✓     |      | 1        | The last element, or a zeroed value. |
| `LastOr`     | ✓      | ✓      | ✓     |      | 1        | The last element, or a default value. |
| `Len`        | ✓      | ✓      | ✓     |      | 1        | Number of elements. |
| `Max`        | ✓      | ✓      |       |      | n        | The maximum value, or a zeroes value. |
| `Median`     |        | ✓      |       |      | n⋅log(n) | Median returns the value separating the higher half from the lower half of a data sample. |
| `Min`        | ✓      | ✓      |       |      | n        | The minimum value, or a zeroed value. |
| `Random`     | ✓      | ✓      | ✓     |      | n        | Select a random element, or a zeroed value if empty. |
| `Reverse`    | ✓      | ✓      | ✓     |      | n        | Reverse elements. |
| `Select`     | ✓      | ✓      | ✓     |      | n        | A new slice containing only the elements that returned true from the condition. |
| `Sort`       | ✓      | ✓      |       |      | n⋅log(n) | Return a new sorted slice. |
| `Sum`        |        | ✓      |       |      | n        | Sum (total) of all elements. |
| `Shuffle`    | ✓      | ✓      | ✓     |      | n        | Returns a new shuffled slice. |
| `Top`        | ✓      | ✓      | ✓     |      | n        | Gets several elements from top(head of slice).|
| `ToStrings`  | ✓      | ✓      | ✓     |      | n        | Transforms each element to a string. |
| `Transform`  | ✓      | ✓      | ✓     |      | n        | A new slice where each element has been transformed. |
| `Unique`     | ✓      | ✓      |       |      | n⋅log(n) | Return a new slice with only unique elements. |
| `Unselect`   | ✓      | ✓      | ✓     |      | n        | A new slice containing only the elements that returned false from the condition. |
| `Values`     |        |        |       | ✓    | n        | Returns all values in the map (in random order). |

# FAQ

## What are the requirements?

`pie` supports many Go versions, all the way back to Go 1.8.

## What are the goals of `pie`?

1. **Type safety.** I never want to hit runtime bugs because I could pass in the
wrong type, or perform an invalid type case out the other end.

2. **Performance.** The functions need to be as fast as native Go
implementations otherwise there's no point in this library existing.

3. **Nil-safe.** All of the functions will happily accept nil and treat them as
empty slices. Apart from less possible panics, it makes it easier to chain.

4. **Immutable.** Functions never modify inputs, unlike some built-ins such as
`sort.Strings`.

## How do I contribute a function?

Pull requests are always welcome.

Here is a comprehensive list of steps to follow to add a new function:

1. Create a new file in the `functions/` directory. The file should be named the
same as the function. You must include documentation for your function.

2. Update `functions/main.go` to register the new function by adding an entry to
`Functions`. Make sure you choose the correct `For` value that is appropriate
for your function.

3. Run `go generate ./... && go install && go generate ./...`. The first
`generate` is to create the pie templates, `install` will update your binary for
the annotations and the second `generate` will use the newly created templates
to update the generated code for the internal types. If you encounter errors
with your code you can safely rerun the command above.

4. If you chose `ForAll` or `ForStructs`, then you must add unit tests to
`pie/carpointers_test.go` and `pie/cars_test.go`.

5. If you chose `ForAll`, `ForNumbersAndStrings` or `ForNumbers`, then you must
add unit tests to `pie/float64s_test.go` and `pie/ints_test.go`.

6. If you chose `ForAll` or `ForStrings`, then you must add unit tests to
`pie/strings_test.go`.

7. If you chose `ForMaps`, then you must add unit tests to `pie/currencies.go`.

8. Update the README to list the new functions.

## Why is the emoji a slice of pizza instead of a pie?

I wanted to pick a name for the project that was short and had an associated
emoji. I liked pie, but then I found out that the pie emoji is not fully
supported everywhere. I didn't want to change the name of the project to cake,
but pizza pie still made sense. I'm not sure if I will change it back to a pie
later.
