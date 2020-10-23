# A guide to Lean Functional Typescript
A guide to Lean Functional Typescript

Overview
========

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

### Pure Functions

**Single argument functions are known as unary functions** 

```typescript

const increment = (num: number) => num + 1

const a = 1
const b = increment(a) // a is still 1, b is 2

```

**Unary functions can be connected together easily to form more powerful functions**

```typescript

const a = 1
const c = increment(increment(a)) // c is 3

```

**fpipe allows you to do the above in a much cleaner way**

_fpipe is modelled after fsharp's pipeline operator_

**install:** `npm i @attack-monkey/fpipe`

```typescript

const a = 1
const c = fpipe(a, increment, increment) // c is 3

```

> `a` is piped into the `increment` function which is piped into the next `increment` function.

**Multi argument functions** use **partial function syntax**

```typescript

const add = (a: number) => (b: number) => a + b

const add3 = add(3)
const seven = add3(4)

```

**Which are easier to use in pipes than regular multi-argument functions**

```typescript

const add = (a: number) => (b: number) => a + b

const seven = fpipe(4, add(3))

```

### Immutable Operations

Pure functions are built from immutable operations, and recursive functions.

**Numbers**

```typescript

const a = 1
const b = a * 10

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

const sum = (numArr: number[], cursor = 0): number =>
    numArr[cursor] + (
        numArr.length - 1 === cursor
            ? 0
            : sum(numArr, cursor + 1)
    )

console.log(
    sum([1, 4, 20])
) // 25

```

There is a lot of doco already out there on immutable operations and recursive functions, so we've only given a tiny taste here.

## Prototype Functions

Javascript / Typescript primitives, arrays and other objects all have methods pre-baked into them.
Some of these methods are mutable (and should not be used within Pure Code).
Some of these methods are immutable and fit within the functional paradigm.
Of particular worth are the Array.prototype functions `map`, `filter`, `reduce`.

We can for example take an array of numbers, and 'map' over them - doubling each number...

```typescript

[1, 2, 3].map(item => item * 2)

```

**Method Chaining**

Method chaining is a little similar to piping.


```typescript


[1, 1.5, 2, 3]
  .map(item => item * 2) // doubles everything to [2, 3, 4, 6]
  .filter(item => item % 2 === 0) // removes any uneven numbers, leaving [2, 4, 6]

```

... And is a common pattern in javascript / typescript

```typescript

  fetch('some url')
    .then(res => res.json())
    .then(doSomethingWithResponse)

```

## Chaining vs Piping

In **Lean** there is a preference towards piping functions together rather than chaining methods. The reason is because methods generally mean a lot of repetition across different classes that implement the same methods. It's possible to break methods into stand alone methods and reuse them - but this is generally a messy process.

In saying that, there are already functional libraries out there that use chaining rather than piping, and even a mix of both chaining and piping.

It is common to see functional libraries that use syntax like the following...

```typescript

const myList = List.of(1, 2, 3)

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

At some stage in your functional programming journey you will embark on what a monad is, and how it works in javascript / typescript.
You will then realise there is this whole complex world of 'lifting' values into 'higher types' that obey 'monad theory'.
Your higher type now gets special methods applied to it, which enables mapping and chaining one higher type to another.
This is all good - but unnecessary in Lean.

In Lean, the focus is on data that meets the call signature of the function.

For example, the two functions below both work on the call signature of `number` therefore any `number` can be passed into these functions. There is no need for an object to be constructed that contains the `increment` and `decrement` method.

```typescript

const increment = (a: number) => a + 1

const decrement = (a: number) => a - 1

```

Since this may lead to a lot of one-off functions being created, it is common to group functions that work on the same call signature into their own object...

```typescript

const increment = (a: number) => a + 1

const decrement = (a: number) => a - 1

// note that Number is already reserved in javascript for the Number Prototype, so here we use Number_

const Number_ = {
  increment,
  decrement
}

const a = 0

const b = Number.decrement(a) // -1

```

Grouping functions like this comes in very handy for functions like `map` which work differently depending on the call signature...

```typescript

const recordMap = <A, B>(f: (a: A) => B) =>
    (r: Record<string, A>) => Object.values
        ? Object.values(r).map(f)
        : Object.keys(r).map(key => r[key]).map(f)

const Record = {
    map: recordMap
}

const arrayMap = <A, B>(f: (a: A) => B) =>
    (a: Array<A>) => a.map(f)

const Array_ = {
    map: arrayMap
}

const myRecord = {
    a: 'cat',
    b: 'dog',
    c: 'monkey'
}

const myArray = ['apple', 'cider', 'vinegar']

fpipe(
    myRecord,
    Record.map(item => item + '!!!'),
    console.log
)

fpipe(
    myArray,
    Array_.map(item => item + '!!!'),
    console.log
)

```

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

Impure Code
===========

In contrast to Pure Code, Impure Code contains mutations, unpredictable results and interactions with things outside of given functions.

Impure Code isn't bad when used appropriately, however most of the time it can be avoided by using Pure Functions and Immutable Operations instead.

State Management
================

There are many services in js / ts land that help manage mutations in a functional way.

Using a State Manager such as Lean-state, Redux, or even Rxjs shifts most of the need for mutation into the hands of a third prty tool - purspose built for handling state. What's more is that these tools allow your code to focus on the 'Pure' domain.

* **Lean State** - https://github.com/attack-monkey/lean-state
* **Reactstate** - https://github.com/attack-monkey/reactstate
* **Redux** - https://redux.js.org/
* **Rxjs** - https://rxjs-dev.firebaseapp.com/

## Conclusion

I hope we've managed to cover the important concepts of Functional Programming without blowing too many brain-cells.

Rather than deep-dive into Mathematical theories, Monads, Functors, etc. **Lean** distills the purity and safety that comes with 
functional programming into a simple and easy-to-apply set of concepts and examples.

Where to from here?

1. Learn how to write immutable operations.

2. Learn how to use recursive functions.

3. While an obvious path is to learn more about other flavors of Functional Programming, remember that **Lean** is about simplicity. It is better (opinion of Lean) to write code that is easily digestible by others than introduce difficult to grasp concepts. **Lean** dove-tails into the javascript / typescript paradigm - where as some other concepts seem foreign to the language. With that caution in mind - learn more about FP.

4. Learn more about functional libraries - including Ramda and Rxjs
