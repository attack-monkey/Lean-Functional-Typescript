# A guide to Lean Functional Typescript

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the concept of Pure Functions and Pure Macros.

Install
=======

While Lean is more conceptual than anything else, the Lean Prelude provides utilities such as **flows** and **pattern matching**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

Lean Principles
===============

1. **Once a variable has been declared - it cannot be changed**

2. **Every impurity can be wrapped in a 'Pure' Macro.** (Note: Any function that does something other than just return a result, is known as a macro in Lean. Functions calculate things. Macros do things).

These two guiding principles form the bedrock of Lean Functional Typescript.

Pure Macros
===========

- When passed the same set of inputs, Pure Macros always return the same result (even if that result is undefined), and therefore appear pure to the currently running function / macro.
- Pure Macros do not mutate anything in any running functions / macros.

In this way `console.log` is a pure macro but `Date.now()` is not.
`Date.now()` is considered impure as it returns a different value each time it is called.

These types of impure macros can be wrapped in a pure macro.
The `now` macro below, takes a pure macro as an argument.
This is known as a Child Macro, for when the parent macro is called, an instance of the Child Macro is spawned, passing in the result of the parent macro.

Eg. The now macro as a pure alternative to Date.now

```typescript

const now = <A>(f: (n: number) => A) => {
    f(Date.now())
}

```

Calling `now` looks like...

```typescript

now(t => console.log('the time is ' + t)) // the time is *whatever the current timestamp is*

```

When `now` gets called it passes the result into the pure macro `t => console.log('the time is ' + t)`
This Child Macro is pure as it always returns undefined
If we were to log `now(t => console.log('the time is ' + t))`, we would see that the return value is also undefined`.

Immutable Operations
====================

In Lean, **once a variable has been declared - it cannot be changed**. In other words all variables should be considered immutable.

So instead of mutating variables it is usually best practice to use immutable operations and pure functions, and assign the result to a new variable.

Eg.

```typescript

const a = { greeting: 'hello ', thing: '' }
const b = { ...a, thing: 'world' }

console.log(a) // hello (i.e. no mutation)
console.log(b) // hello world

```

Mutables
========

Mutability can however be managed in a completely pure way - by abiding by the rules of Pure Macros.
`pureMutable` takes a starting value along with a Pure Macro - let's call this the 'Child Macro'
It spawns a new instance of the Child Macro passing in the initial value as input, as well as a special `set` macro.
When `set` is called, nothing is mutated in any currently running macros, BUT a new Child Macro is spawned, passing in the new value as input.
Actually both the old and new value are passed into the Child Macro should it need to work with the old and new.

```typescript

pureMutable(1)((_, newValue, set) => {
    wait(1000)(() => {
        console.log(newValue)
        set(newValue + 1)
    })
})

```

So `PureMutable` abides by the rules of Pure Macros because it appears as a pure function to the currently running function / macro. The trick is that it spawns new running instances of the Child Macro, passing new values into it.

Note that PureMutable also comes with some safe-guards. 
1. `set` can only be called if a time buffer is in place between running the `pureMutable` and calling `set`. This prevents inifnite looping.
2. `set` is only callable once per running instance of the Child Macro. This stops multiple instances running at the same time and forces a safer way of working with data.

```typescript

pureMutable(1)((_, newValue, set) => {
    set(newValue + 1) // This will cause an error because set cannot be called synchronously
    wait(1000)(() => {
        console.log(newValue)
        set(newValue + 1) // This set is ok because there is a time buffer between the pureMutable and the set.
        set(newValue + 2) // This set is not ok, because the previous set would have already been triggered.
    })
})

```

`PureMutable` is available in the **Lean Prelude**

Working with PureMutable can take some time to get used to. It's best to use it in terms of scope. A global scope `pureMutable` can exist outside of the main app function. Then layers of lower level scoped `pureMutable`'s can be used in smaller components.

Pure Functions
==============

Pure Functions are the building blocks of any functional programming. Pure Functions take in input and return output - without mutating anything.

This therefore enforces data that is immutable.
  
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

### Pipes and Flow

**Pipes allow you to do the above in a much cleaner way**

In FP languages such as F# you can write the above as:

```fsharp

let a = 1 // equivalent to js const

let c = a |> increment |> increment

```

> `a` is piped into the `increment` function which is piped into the next `increment` function.

and while there is not yet (though there is a proposal for this) a pipeline operator in js / ts, Lean provides something more powerful.

**A `flow` is able to pipe a value through a series of functions, but also allows for async flows, conditional flows, and streams**

For now though, think of flows as a way to achieve piping.

```typescript

const three = flow(1)
  .then(increment)
  .then(increment)
  .done()

```

**We'll get to the more powerful side of flows later on**

**Most of the time in Lean, we use partial function syntax to write functions...**

A partial function is a function that returns another function, and so on, until the full function is played out.

```typescript

const add = (a: number) => (b: number) => a + b

const add3 = add(3)
const seven = add3(4)

```

**Which are easier to use in flows than regular multi-argument functions**

```typescript

const add = (a: number) => (b: number) => a + b

const seven = flow(4).then(add(3))

```

**Partials also provide a simple, powerful way of doing Dependency Injection**

```typescript

type LoggingTool = (a: any) => void

const simpleLog: LoggingTool = a => console.log(a)

const logInitialiser = (loggingTool: LoggingTool) => (a: any) => loggingTool(a)

// Set up our logging tool...

const log = logInitialiser(simpleLog)

// Now we can log

log('hello world')

// In production, we may swap out simpleLog for something more sophisticated - but we don't need to change our log statements.


```

Pure Functions can also be recursive and should be used in place of loops - because loops mutate values.

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

If however `reverse` and `append` are simply stand alone functions, then we can use flows to connect the functions together.

```typescript

const myNewList =
  flow([1,2,3])
    .then(reverse)
    .then(append('blast off!'))
    .done()

```

None of the functions are 'bound' to `this` in a class / constructor, and instead can work on any values that meet their call signature. Simplicity and flexibility are baked in. **This is major part of what Lean is all about**

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

const PureNumber = {
  increment,
  decrement
}

const a = 0

const b = PureNumber.decrement(a) // -1

```

Grouping functions like this comes in very handy for functions like `map`, `filter`, and `reduce` which work differently depending on the call signature...

```typescript


const myArray = ['apple', 'cider', 'vinegar']

const myRecord = {
    a: 'cat',
    b: 'dog',
    c: 'monkey'
}

flow(myArray)
  .then(Array_.map(item => item + '!!!'))
  .then(Array_.filter(item => item !== 'vinegar!!!'))
  .then(Array_.reduce((ac: string, cv: string) => ac + cv)('I like '))
  .then(console.log)

flow(myRecord)
  .then(Record.map(item => item + '!!!'))
  .then(Record.filter(item => item !== 'monkey!!!'))
  .then(Record.reduce((ac: string, cv: string) => ac + cv)('hello '))
  .then(console.log)

```

> Note that `Record` and `Array_` are provided in the Prelude.

Even though the `Array_` and `Record` library have a set of methods, the data being passed into these methods are not bound to those methods.
For example, there may be a different library that acts on just number arrays - let's call it `NumberArray`. An array of numbers would therefore be able to be passed into any of the methods from both `Array_` and `NumberArray` and may look something like...

```typescript

flow([1, 2, 3])
  .then(Array_.map(item => item * 2))
  .then(NumberArray.sum)
  .then(console.log)

```

## If you do need to write functional classes... you can do so without mutating properties...

It's possible to write classes that don't mutate properties and that still achieve chainability.

In this example, rather than mutate the property `a`, the `pipe` method returns a new `Pipe` object altogether.

This is a simple pipe emulator...

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

pipe('hello world')
  .pipe(console.log)

```

## Pattern Matching

In functional languages like F# most if / then / else style logic is handled through Pattern Matching.

Lean also provides pattern matching.

Here it's possible to create a type that can be matched against at run-time, and based on that match, trigger a function.

```typescript

match('hello')
  .with_($string, s => console.log(s + ' world'))
  .with_($unknown, _ => console.log('unable to match'))
  .done()
  
```

Pattern matching uses the matcha-match library, and is worth getting well aquainted with - as it provides powerful, clean ways of handling logic.

Read more at [docs](https://www.npmjs.com/package/matcha_match)

> Note that `match` is a wrapper around matcha_match's `patternMatch` function.


## Conclusion

- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create complex data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables
- Pure Macros 'do something' as well as return a value. When dealing with impure values, Pure Macros spawn new instances of Child Macros and pass impure values as inputs. They do this instead of returning an impure value inside a pure function which would otherwise pollute the otherwise pure function.

