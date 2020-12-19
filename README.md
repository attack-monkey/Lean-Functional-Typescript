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

We recommend typescript for the type-safety that it gives.

Pure Code
=========

Lean has a heavy emphasis on writing Pure Code where ever possible.

Pure Code is broken into two main concepts:
  1. Pure Functions
  2. Pure Macros

Pure Functions are functions / operations that:
  - Don't mutate anything.
  - Only interact with inputs, constants, and other 'pure' functions to derive a result.
  - When passed the same set of inputs, always return the same result.
  
Where Pure Functions are core to any Functional Programming, Pure Macros are a Lean Concept. They are very similar to Pure Functions except that they also contain side effects.
That is, they give instructions to things outside of the Pure Function, and therefore change the world around them.
They still only use inputs, constants, and other 'pure' functions ( or 'pure' macros ) to derive a result.
They still return the same result, when passed the same set of inputs.

### Pure Functions

#### A note on immutability

One of the guiding principles of Pure Functions is that every variable (aka Value) is immutable.
This in itself stops many many bugs from ever entering production. Bugs from mutability often arise when one object references another object. One of those objects is then mutated - mutating the other accidentally.

Using the `const` keyword to declare variables ensures that they cannot be reassigned a new value after declaration.
This makes strings, numbers, and booleans effectively immutable - but doesn't do the same for objects / arrays.
Since `const` only makes a variable non-reassignable, it's still possible to mutate an objects properties and methods.

The Lean Prelude provides `freeze` which can be wrapped around an object / array to ensure that the objects keys cannot be reassigned another value.
If however the object contains deeper objects below - then they also need to be wrapped ...

```typescript

const a = freeze({
  first: freeze({
    second: 'protected'
  })
})

```

Providing that people follow Lean correctly, there is no real need to use `freeze`, but it is there should extra protection be required.

#### Now back to Pure Functions

Pure Functions are the building blocks of Pure Code. Pure Functions take in input and return output - without ever mutating the input.

This therefore enforces data that is immutable.

> ^^ We want to write code like this as much as possible.

Pure Functions:
  - Don't mutate anything.
  - Only interact with inputs, constants, and other 'pure' functions to derive a result.
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

`fpipe` achieves the effect of piping by using a 'variable argument function' - but it is limited. It can only pipe through up to 10 functions.

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

This is totally fine... but it means that `reverse` and `append` are bound to the `List` class. If those methods are then to be used in another class, we have to either write those methods again, or use class extensions, or messy mixins.

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

**In Lean, the focus is on data that meets the call signature of the function.**

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

Impure Code
===========

In contrast to Pure Code, Impure Code contains mutations, unpredictable results and interactions with things outside of given functions.

Impure Code isn't bad when used appropriately, however most of the time it can be avoided by using Pure Functions and Immutable Operations instead.

In Lean, any function that does something other than just return a result, is known as a macro. Functions calculate things. Macros do things.

**Common in-built macros**

- `console.log` for example fits the description of macro, since it is a function that interacts with the machine's / brower's console.
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

Pure Macros:
- Only interact with inputs, constants, and other Pure Functions ( and Pure Macros ) to derive a result.
- When passed the same set of inputs, always return the same result.

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

A Mutable is a safe store of mutable data.

They use the lean-mutable library (coming soon).

The `mutable` macro wraps a value, turning it into a Mutable and then fires the Mutable into an awaiting pure macro.
The Mutable is unreachable outside of the scope of the mutable macro.

To update the value of a mutable use `set`, which takes the Mutable ( + an optional pass-in value ), along with a pure function to produce a new state.

Use `unwrap` to access the current state.

```typescript

mutable(1)(counter => {
    set(counter)(v => v + 1) // update to 2
    set(counter)(v => v + 1) // update to 3
    unwrap(counter)(v => {   // unwrap
        console.log(v)       // logs 3
    })
                             // goes out of scope - no longer reachable
})

```

The `mutable`, `unwrap`, and `set` macros are all 'pure' macros - but not 'pure' functions.
By definition a macro 'does' something other than return a value, and in the case of set - it's mutating state.
The 'Mutable' makes working with mutable state safer, since unwrapped state is always passed to an accompanying pure macro, and updates always occur using pure functions.

Mutables can also be subscribed to.
Subscriptions must have id's which prevent memory leaks associated with id-less subscriptions.
The id can be used later to tear down a subscription with destroy.
Subscriptions also have a safeguard whereby a set cannot be called during the emit phase of the set / subscribe - which prevents infinite loops.

```typescript

mutable(1)(counter => {
    subscribe('s1')(counter)(v => { // s1 is the Id that we are assigning to our subscription.
        console.log(`value is now ${v}`)
        set(counter)(v => v + 1) // This will throw an error because setting during the emit phase will otherwise cause infinite loops.
    })
    set(counter)(v => v)     // set with current
    set(counter)(v => v + 1) // update to 2
    set(counter)(v => v + 1) // update to 3
                             // goes out of scope - no longer reachable
})

```

The following shows how to create a well described pure macro for a Mutable to be passed into.
The pure macro can then be tested easily.

```typescript

type Person = {
    name: string
    age: number
}
type People = Record<string, Person>

const init: People = {
    person1: {
        name: 'Ben',
        age: 39
    },
    person2: {
        name: 'Sherri',
        age: 30
    }
}

const pureMacro = (people: Mutable<People>) => {

    const op1 = subscribe('people-sub-1')(people)(v => {
        Object.keys(v).forEach(key => {
            console.log(`hello ${v[key].name}, you are ${v[key].age}`)
        })
    })

    const op2 = set(people)(v => ({...v, person1: { name: v.person1.name, age: v.person1.age + 1 } }))

    return {
        macro: 'pureMacro',
        params: [people],
        subMacros: [
            op1,
            op2
        ]
    }
}

```

It's easy to test that the pureMacro is performing correctly by logging it's response.
Since it's response describes what it is doing, we can ensure that it is behaving as expected.
To create a stub Mutable for passing into the pureMacro (for testing) - use `mutableStub`.

test...

```typescript

console.log(`test\n`, pureMacro(mutableStub(init)))

```

Run for real...

```typescript

mutable(init)(pureMacro)

```


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
