# A guide to Lean Functional Typescript
A guide to Lean Functional Typescript

## Overview

There is no one way to 'do' functional programming (FP) in a language like typescript, so think of **Lean** as a particular flavor. 

Other variants are often very complex and not very idiomatic to Javascript / Typescript.

In **Lean**, the focus is on **simple code** that is pure, typesafe, and immutable.

**Lean** deliberately avoids many hardcore functional concepts because they often complicate things for teams.

Instead **Lean** keeps things simple and actually feeling like Javascript / Typescript.

So... here goes...

Pure Code
=========

Pure Functions are the building blocks of Pure Code. Pure Functions take in input and return output - without ever mutating the input.

This therefore enforces data that is immutable.

> ^^ We want to write code like this as much as possible.

Pure Functions:
  - Don't mutate anything.
  - Only interact with inputs, constants, and other 'pure' functions to derive a result.
  - When passed the same set of inputs always return the same result.

Pure functions are built from immutable operations, and recursive functions.

### Some Immutable Operations...

**Numbers**

```typescript

const a = 1
const b = a *10

```

**Strings**

```typescript

const a = 'hello'
const b = a + ' world'

```

**Arrays**

```typescript

const a = [1,2,3]
const b = [...a, 4,5,6]

```

**Objects**

```typescript

const person1 = {
  name: 'ben',
  isType: 'person'
}

const person2 = {
  ...a,
  name: 'jan'
}

```

### An example of a recursive function

```typescript

const recFn = (a = 0) =>
  number > 50
    ? console.log('50!!!')
    : recFn(a + 1)

recFn()

```

There is a lot of doco already out there on immutable operations and recursive functions, so we've only given a tiny taste here.

Impure Code
===========

In contrast to Pure Code, Impure Code contains mutations, unpredictable results and interactions with things outside of given functions.

Impure Code isn't bad when used appropriately, however most of the time it can be avoided by using Pure Functions and Immutable Operations instead.

Functions over Classes + Methods
================================

Classes bind specific methods to an object, which more often than not mutate the object's properties.
This not only makes class + method syntax impure - but it also locks methods against objects.

Pure Functions by contrast have their 'properties' passed in, and can be used on anything as long as the properties meet the 'call signature' of the function.
This flexibility allows the result of one function to be passed to another, and so on.
This is known as Functional Composition and is a pretty big deal.

Functional Composition is used in place of where you would otherwise find method chaining.

Pipes & Functional Composition
==============================

Pipes take the output of one function and pass it into the input of the next function, until a result is generated.
Using pipes to build complex functions out of simpler ones is known as functional composition.

eg. 

```typescript

// Here we are using the fpipe library which is modelled after fsharp's pipeline operator `|>`

const increment = (a: number) => a + 1 
const three = fpipe(1, increment, increment)

// This is the same as calling

const three = increment(
  increment(
    1
  )
)

```

To use `fpipe`...

`npm i @attack-monkey/fpipe`

`fpipe` first takes a value, followed by a series of Unary (single argument) functions. The value is passed into the first function. The result is passed into the second function and so on.

```typescript

const addTwo = (a: number) => fpipe(a, increment, increment)
const three = addTwo(1)

```

To use `fpipe` requires functions to be Unary, but often functions require more than one argument.

Enter partial functions.

Unary Functions & Partial Functions
===================================

Partial functions use 'closures' to trap state and return a function with that trapped state inside.
When the 'returned function' is then called - the 'full function' completes.

eg.

```typescript

const add = (a: number) => (b: number) => a + b

const plus6 = add(6)

console.log(plus6(1)) // 7
console.log(plus6(6)) // 12

```

This technique allows multi-argument functions to be used in `pipes`

```typescript

console.log(
  fpipe(
    1,
    add(3),
    add(2)
  )
)

```

^^ So here 1 is passed to add(3), which makes 4, which is then passed to add(2), which makes 6

# Prototype Functions

Javascript / Typescript primitives, arrays and other objects all have methods pre-baked into them.
Some of these methods are mutable (and should not be used within Pure Code).
Some of these methods are immutable and fit within the functional paradigm.
Of particular worth are the Array.prototype functions `map`, `filter`, `reduce`.

We can for example take an array of numbers, and 'map' over them - doubling each number...

```typescript

[1, 2, 3].map(item => item * 2)

```

While these functions are great to be aware of, it might also lead you to think - hey I could write my own classes with functional methods.

```typescript

class MyCoolArray {
    private privateValue: number[]
    constructor(value: number[]) {
        this.privateValue = value
    }
    doubleMap() {
        this.privateValue = this.privateValue.map(item => item * 2)
        return this
    }
    value() {
        return this.privateValue
    }
}

const a = new MyCoolArray([1, 2, 3])
const b = a.doubleMap().doubleMap()
console.log(b.value())

```

However in writing the above class, you are mutating a property and locking methods to an object.

Instead, the following is preferred:

```typescript

const doubleMap = (value: number[]) => value.map(item => item * 2) // still making use of the built in `.map`
const quadMap = (x: number) => fpipe(x, doubleMap, doubleMap)

const a = [1, 2, 3]
const b = quadMap(a)

```

And `doubleMap` and `quadMap` can be added to a parent object to help with grouping.

```typescript

import { doubleMap } from '...'
import { quadMap } from '...'

export const NumberArray = {
  doubleMap,
  quadMap
}

```

```typescript

import { NumberArray } from '...'

const a = NumberArray.quadmap([1, 2, 3])

```

> The parent object is used to group functions that work on the same call signature

All in all - less code and more flexibility :D

## Chaining vs Piping

In **Lean** there is a preference towards piping functions together rather than chaining methods. The reason is because methods generally mean a lot of repetition across different classes that implement the same methods. It's possible to break methods into stand alone methods and reuse them - but this is generally a messy process.

In saying that, there are already functional libraries out there that use chaining rather than piping, and even a mix of both chaining and piping.

It is common to see functional libraries that use syntax like the following...

```typescript

const myList = List.of([1, 2, 3])

myList
  .reverse()
  .append('blast off!')

```

This is totally fine... but when writing your own functions - Lean prefers...

```typescript

fpipe(
  [1, 2, 3],
  reverse,
  append('blast off!')
)

```

Why? Because none of the functions are 'bound' to an object or class, but instead can work on any values that meet their call signature. Simplicity and flexibility is baked in.



## Pattern Matching

In functional languages like F# most if / then / else style logic is handled through Pattern Matching.

Using 'Matcha', similar pattern matching of complex objects is possible in Javascript / Typescript.

```

npm i matcha_match

```

Here it's possible to create a type that can be matched against at run-time, and based on that match, trigger a function.

Notice how the compile-time `Cat` type is actually built from the run-time type.

The patternMatch 'with_' infers the type onto the matched object - so the `doCatThing` function gets a value guaranteed to be of type `Cat`.

```typescript

import { patternMatch, with_ } from 'matcha_match/lib/index'
import { $string } from 'matcha_match/lib/runtime-interfaces/$string'
import { $literal } from 'matcha_match/lib/runtime-interfaces/$literal'
import { $unknown } from 'matcha_match/lib/runtime-interfaces/$unknown'

const $runtimeCatType = {
  animal: $literal<'cat'>('cat'),
  name: {
    first: $string,
    last: $string
  }
}

type Cat = typeof $runtimeCatType

const myCat = {
  animal: 'cat',
  name: {
    first: 'felix',
    last: 'the cat'
  }
}

const doCatThing = (cat: Cat) => {
  console.log(`meow... my name is ${cat.name.first}`)
}

patternMatch(
  myCat,
  with_($runtimeCatType, doCatThing),
  with_($unknown, _ => console.log('not a cat'))
)

```

## Conclusion

I hope we've managed to cover the important concepts of Functional Programming without blowing too many brain-cells.

Rather than deep-dive into Mathematical theories, Monads, Functors, etc. **Lean** distills the purity and safety that comes with 
functional programming into a simple and easy-to-apply set of concepts and examples.

Where to from here?

1. Learn how to write immutable operations.

2. Learn how to use recursive functions.

3. While an obvious path is to learn more about other flavors of Functional Programming, remember that **Lean** is about simplicity. It is better to write code that is easily digestible by others than introduce difficult to grasp concepts. **Lean** dove-tails into the javascript / typescript paradigm - where as some other concepts seem foreign to the language. With that caution in mind - learn more about FP.

4. Learn more about functional libraries - including Ramda and Rxjs
