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

Install
=======

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

It works with both javascript and typescript...

Node (Javascript using `require`)

```javascript

const { match, $string, $unknown } = require('@attack-monkey/lean-f-ts-prelude')

match('hello')
  .with_($string, s => console.log(s + ' world'))
  .with_($unknown, _ => console.log('unable to match'))
  .done()
  
```

Typescript / Javascript using `import`

```typescript

import { match, $string, $unknown } from '@attack-monkey/lean-f-ts-prelude'

match('hello')
  .with_($string, s => console.log(s + ' world'))
  .with_($unknown, _ => console.log('unable to match'))
  .done()
  
```

We recommend typescript for the type-safety that it gives.

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

### Pipes

**Pipes allow you to do the above in a much cleaner way**

In FP languages such as F# you can write the above as:

```fsharp

let a = 1 // equivalent to js const

let c = a |> increment |> increment

```

> `a` is piped into the `increment` function which is piped into the next `increment` function.

and while there is not yet (though there is a proposal for this) a pipeline operator in js / ts, Lean does provide some equivalents.

**fpipe is modelled after F#'s pipeline operator**

```typescript

const a = 1
const c = fpipe(a, increment, increment) // c is 3

```

`fpipe` achieves the effect of piping by using a 'variable argument function' - but it is limited to up to 10 functions.

**pipe achieves the same effect by using a object / method approach and while is more verbose - is not limited**


```typescript

const a = 1
const c = pipe(a).pipe(increment).pipe(increment).done() // c is 3

```

Which one is better? 

It doesn't really matter. `fpipe` is leaner in most situations, but `pipe` will work in all situations.

**Most of the time in Lean, we use partial function syntax to write functions...**

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

Method chaining works by calling a method on an object, which produces a result. The result itself has methods, which can then be called, and so on.

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

Infact `pipe` uses method-chaining to pipe the result of one function into the next pipe.

```typescript

  pipe(1)
    .pipe(increment)
    .pipe(increment)

```

## Chaining vs Piping

It is common to see functional libraries provide classes that 'lift' values to provide methods that then work on the value or collection of values inside.

```typescript

const myList = List.of(1, 2, 3)

const myNewList = myList
  .reverse()
  .append('blast off!')
  .done()

```

This is totally fine... but it means that `reverse` and `append` are bound to the `List` class. If those methods are then to be used in another class, we have to use class extensions, repeated code, or messy mixins.

If however `reverse` and `append` are simply stand alone functions, then we can use pipes to connect the functions together.

```typescript

const myNewList = 
  fpipe(
    [1, 2, 3],
    reverse,
    append('blast off!')
  )

```

or

```typescript

const myNewList =
  pipe([1,2,3])
    .pipe(reverse)
    .pipe(append('blast off!'))
    .done()

```

None of the functions are 'bound' to `this` in a class / constructor, and instead can work on any values that meet their call signature. Simplicity and flexibility is baked in.

In Lean, the focus is on data that meets the call signature of the function.

For example, the two functions below both work on the call signature of `number` therefore any `number` can be passed into these functions. There is no need for an object to be constructed from a class containing the `increment` and `decrement` method. There is no need to bind to an object's `this`

```typescript

const increment = (a: number) => a + 1

const decrement = (a: number) => a - 1

```

Since this may lead to a lot of one-off functions being created, it is common to group functions that work on the same call signature into their own library object...

```typescript

const increment = (a: number) => a + 1

const decrement = (a: number) => a - 1

// note that Number is already reserved in javascript for the Number Prototype, so here we use Number_

const Number_ = {
  increment,
  decrement
}

const a = 0

const b = Number_.decrement(a) // -1

```

Grouping functions like this comes in very handy for functions like `map`, `filter`, and `reduce` which work differently depending on the call signature...

```typescript

// Array_

const arrayMap = <A, B>(f: (a: A) => B) =>
    (a: Array<A>) => a.map(f)

const arrayFilter = <A, B>(f: (a: A) => B) =>
    (a: Array<A>) => a.filter(f)

const arrayReduce = <A, B>(f: (ac: B, cv: A, i?: number) => B) => (init: B) =>
    (a: Array<A>): B => a.reduce(f, init)

const Array_ = {
    map: arrayMap,
    filter: arrayFilter,
    reduce: arrayReduce
}

// Record

const recordMap = <A, B>(f: (a: A) => B) =>
    (r: Record<string, A>): Record<string, B> =>
        Object.keys(r).reduce((ac, key) => ({ ...ac, [key]: f(r[key]) }), {} as Record<string, B>)

const recordFilter = <A>(f: (a: A) => boolean) =>
    (r: Record<string, A>): Record<string, A> =>
        Object.keys(r).filter(key => f(r[key])).reduce((ac, key) => ({ ...ac, [key]: r[key] }), {} as Record<string, A>)

const recordReduce = <A, B>(f: (ac: B, cv: A, i?: number) => B) => (init: B) =>
    (r: Record<string, A>): B =>
        Object.keys(r).map(key => r[key]).reduce(f, init)

const Record = {
    map: recordMap,
    filter: recordFilter,
    reduce: recordReduce
}

const myArray = ['apple', 'cider', 'vinegar']

const myRecord = {
    a: 'cat',
    b: 'dog',
    c: 'monkey'
}

fpipe(
    myArray,
    Array_.map(item => item + '!!!'),
    Array_.filter(item => item !== 'vinegar!!!'),
    Array_.reduce((ac: string, cv: string) => ac + cv)('I like '),
    console.log
)

fpipe(
    myRecord,
    Record.map(item => item + '!!!'),
    Record.filter(item => item !== 'monkey!!!'),
    Record.reduce((ac: string, cv: string) => ac + cv)('hello '),
    console.log
)

```

> Note that `Record` and `Array_` are provided in the Prelude.

Even though the `Array_` and `Record` library have a set of methods, the data being passed into these methods are not bound to those methods.
For example, there may be a different library that acts on just number arrays - let's call it `NumberArray`. An array of numbers would therefore be able to be passed into any of the methods from both `Array_` and `NumberArray` and may look something like...

```typescript

fpipe(
    [1, 2, 3],
    Array_.map(item => item * 2),
    NumberArray.sum
    console.log
)

```

## If you do need to write functional classes... you can do so without mutating properties...

It's possible to write classes that don't mutate properties and that still achieve chainability.

Rather than mutate the property `a`, the `pipe` method returns a new `Pipe` class altogether.

```typescript

class Pipe <A> { 
    private a: A
    static of = <A>(a: A) => new Pipe(a)
    constructor (a: A) {
        this.a = a
    }
    pipe = <B>(f: (a: A) => B) => Pipe.of(f(this.a))
    done = () => this.a
}

const pipe = Pipe.of

```

## Pattern Matching

In functional languages like F# most if / then / else style logic is handled through Pattern Matching.

Lean also provides pattern matching.

Here it's possible to create a type that can be matched against at run-time, and based on that match, trigger a function.

Notice how the compile-time `Cat` type is actually built from the run-time type.

The patternMatch 'with_' infers the type onto the matched object - so the `doCatThing` function gets a value guaranteed to be of type `Cat`.

```typescript

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

fmatch(
  myCat,
  with_($runtimeCatType, doCatThing),
  with_($unknown, _ => console.log('not a cat'))
)

```

Like `pipe` / `fpipe`, Lean provides both `match` / `fmatch`.

`fmatch` is limited to 5 `with_` handles where as `match` is unlimited.

**Example using `match`**

```typescript

match('hello')
  .with_($string, s => console.log(s + ' world'))
  .with_($unknown, _ => console.log('unable to match'))
  .done()
  
```

Pattern matching uses the matcha-match library, and is worth getting well aquainted with - as it provides powerful, clean ways of handling logic.

Read more at [docs](https://www.npmjs.com/package/matcha_match)

> Note that Lean calls matcha-match's `patternMatch` function `fmatch`, and provides the 'object / method' variant of `match`.

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
