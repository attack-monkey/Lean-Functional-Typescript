# A guide to Lean Functional Typescript
A guide to Lean Functional Typescript

Overview
========

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the concept of Pure Functions and Pure Macros.

It is less 'Haskell' inspired and more ML / F# inspired which is far easier to grasp and makes writing large scale applications a breeze.

Install
=======

While Lean is more conceptual than anything else, there are utility modules to bring some of the concepts alive.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

Pure Code
=========

Pure Code is broken into three main concepts:
  1. Immutable Operations
  2. Pure Functions
  3. Pure Macros

**Immutable Operations**

Immutable Operations treat variables as immutable values - that cannot be changed once declared. **This is the Golden Rule of Lean**.
Instead of mutating values, create new values using the previous value as a starting point.

Eg.

```typescript

const a = { greeting: 'hello ', thing: '' }
const b = { ...a, thing: 'world' }

console.log(a) // hello (i.e. no mutation)
console.log(b) // hello world

```

Immutable Operations can also be recursive and are generally preferred to loops - because loops mutate values.

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

**Pure Functions**

Pure Functions:
  - Don't mutate anything.
  - When passed the same set of inputs, always return the same result.

**Pure Macros**

Where Pure Functions are core to any Functional Programming, Pure Macros are a Lean Concept. 
They are very similar to Pure Functions except that they also contain side effects.
That is, they give instructions to things outside of the Pure Function, and therefore change the world around them.
They still return the same result, when passed the same set of inputs.

### Pure Functions

#### A note on immutability

One of the guiding principles of Pure Functions is that every variable (aka Value) is immutable.
This in itself stops many many bugs from ever entering production. Bugs from mutability often arise when one object references another object. One of those objects is then mutated - mutating the other accidentally.

Using the `const` keyword to declare variables ensures that they cannot be reassigned a new value after declaration.
This makes strings, numbers, and booleans effectively immutable - but doesn't do the same for objects / arrays.
Since `const` only makes a variable non-reassignable, it's still possible to mutate an objects properties and methods.

Providing the Lean Golden rule is followed - then this is no problem.

**Golden Rule**

Variables must be treated as immutable values - that cannot be changed once declared.

#### Now back to Pure Functions

Pure Functions are the building blocks of Pure Code. Pure Functions take in input and return output - without ever mutating the input.

This therefore enforces data that is immutable.

> ^^ We want to write code like this as much as possible.

Pure Functions:
  - Don't mutate anything.
  - When passed the same set of inputs always return the same result.
  
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

`fpipe` achieves the effect of piping by using a function that takes a variable number of arguments - but it is limited. It can only pipe through up to 10 functions.

**pipe achieves the same effect and while is more verbose - is not limited**


```typescript

const a = 1
const c = pipe(a).pipe(increment).pipe(increment).done() // c is 3

```

Which one is better? 

It doesn't really matter. `fpipe` is leaner in most situations, but `pipe` will work in all situations.

**Also ... notice how Promises are a form of async pipe**

```typescript

somePromise
  .then(increment)
  .then(increment)

```

**And Lean Prelude also provides `flow` which is more powerful than promises ( and can also be sync / async - so can be used instead of `pipe` )**

```typescript

flow(1)
  .then(increment)
  .then(increment)
  .done()

```

**We'll get to flows later on**

**Most of the time in Lean, we use partial function syntax to write functions...**

A partial function is a function that returns another function, and so on, until the full function is played out.

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

This is OK... but is not Lean. It means that `reverse` and `append` are bound to the `List` class. If those methods are then to be used in another class, we have to either write those methods again, or use class extensions, or messy mixins.

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

None of the functions are 'bound' to `this` in a class / constructor, and instead can work on any values that meet their call signature. Simplicity and flexibility are baked in. **This is what Lean is all about**

**In Lean, the focus is on data that meets the call signature of the function.**

For example, the two functions below both work on the call signature of `number` therefore any `number` can be passed into these functions. There is no need for an object to be constructed from a class containing the `increment` and `decrement` method. There is no need to bind to an object's `this`

```typescript

const increment = (a: number) => a + 1

const decrement = (a: number) => a - 1

```

It is common to group functions that work on the same call signature into their own library object...

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

Rather than mutate the property `a`, the `pipe` method returns a new `Pipe` object altogether.

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

The `fmatch` `with_` infers the type onto a matched object - so the `doCatThing` function gets a value guaranteed to be of type `Cat`.

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

> Note that Lean calls matcha-match's `patternMatch` function `fmatch`, and `match` is also a wrapper around `patternMatch`.

Pure Macros
===========

In Lean, any function that does something other than just return a result, is known as a macro. Functions calculate things. Macros do things.

**Common in-built macros**

- `console.log` fits the description of macro, since it is a function that interacts with the machine's / brower's console.
- `Date.now()` interacts with the machine to get the current timestamp.
- `Math.random` returns a random number value - which is impure.
- `setTimeout` / `setInterval` interacts with the event-loop.

**Macros also include:**

- SDK's that interact with Databases
- The `fetch` api that interacts with outside services
- Anything to do with the DOM
- Anything that interacts with the file system or browser storage
- Anything that interacts with mutable state

So it's fair to say that macros are everywhere.

**Lean makes the bold claim that every impurity can be wrapped in a 'Pure' Macro.**

There are lots of Javascript / Typescript Libraries out there that are not pure and that require wrapping by Pure Macros.
This impure code for the most part is isolated in your node_modules folder - but if you have in-house impure libraries - then it is best practice to separate this impure code from the Pure Lean code. Pure Macros interface with / wrap this impure code to allow the main application run in a pure way.

Pure Macros:
- When passed the same set of inputs, always return the same result (even if that result is undefined) and therefore appear pure to the currently running function / macro.
- Do not mutate anything in any running functions / macros.

However they also send instructions to the 'outside world' to perform tasks, and get information from the 'outside world'.

A pure macro is a function that does something and returns a pure response - even if that response is undefined.
In this way `console.log` is a pure macro but `Date.now()` is not.
`Date.now()` is considered impure as it returns a different value each time it is called.

When creating pure macros it is best practice to have them do a single thing and that the response return a description of the macro and the params being passed to it.
In this way, the macro can be 'black-box' tested where we effectively say - whatever happens in the box is the domain of that box, but we can easily test that we have:
1. called the box
2. passed it the expected params

It is also good practice to pass back whether an error occurred and what sub-macros (if any were also called).

Let's take a look at Date.now() as this is something that always produces a different and therefore impure result.

These types of impure macros can be wrapped in a pure macro - which in turn allows the surrounding macro to be pure.
The now-macro below, takes a pure macro as an argument.
When the outer macro fires, the inner pure macro is called with the result of the outer macro.

Eg. The now function as a pure alternative to Date.now

```typescript

const now = <A>(f: (n: number) => A) => {
    f(Date.now())
    return {
        macro: 'now'
    }
}

```

When `now` gets called it passes the result into the pure macro `t => console.log('the time is ' + t)`
This inner macro is pure as it always returns undefined
If we were to log `now(t => console.log('the time is ' + t))`, we would see that the return value is always `{ macro: 'now' }` which is nice because it describes the macro being called.

```typescript

now(t => console.log('the time is ' + t))

```

Mutables (LIBRARIES COMING SOON)
========

Mutability can also be managed in a completely pure way.
pureMutable takes a starting value along with a Pure Macro - let's call this the 'Child Macro'
It spawns a new instance of the Child Macro passing in the initial value as input, as well as a special 'set' macro.
When 'set' is called, nothing is mutated in any currently running macros, BUT a new Child Macro is spawned passing in the new value as input.
Actually both the old and new value are passed into the Child Macro should it need to work with the old and new. 
Due to the potential for infinite looping, there MUST be a time buffer between the Child Macro being run and set being called.

```typescript

pureMutable(1)((_, newValue, set) => {
    set(newValue + 1) // This will throw an error as `set` cannot be called without some sort of 'time buffer'
    setTimeout(() => {
        console.log(newValue)
        set(newValue + 1) // This is ok because the setTimeout causes a time-buffer.
    }, 1000)
})

```

## Conclusion

- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create complex data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables
- Pure Macros 'do something' as well as return a value. When dealing with impure values, Pure Macros spawn new instances of Child Macros and pass impure values as inputs. They do this instead of returning an impure value inside a pure function which would otherwise pollute the otherwise pure function.

