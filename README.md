# A guide to Lean Functional Typescript

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the the use of Pure Functions and Abstracted Effects.

Install
=======

While Lean is more conceptual than anything else, the Lean Prelude provides utilities such as **pattern matching**, **piping**, and **flows**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

`import { pipe, match } from '@attack-monkey/lean-f-ts-prelude/dist/src'`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

The Lean Rule
===============

Effects should be piped into pure-functions in an unprocessed state, so that pure-functions can manage any impurities in a pure way.

```typescript

// Lean separates effects from pure-functions, allowing logic and effect modelling to be tested in a completely pure way.

// effects are abstracted from their primitive effects, for possible extensions
const rnd = Math.random
const log = console.log

// pure-functions are used to process passed in value-producers and other data
type abc = [ () => number, () => number, (...data: any[]) => void | LogEffect]
const main = ([a, b, c]: abc) => c(a() + b())

// effects are passed in unprocessed, and piped forward into pure functions
pipe([rnd, rnd, log] as abc)
  .pipe(main)

// For testing, effects can be modelled...

type LogEffect = ["Log", number]

const testLog = (...data: any[]) => {
  const dataOut = data[0] as number
  return ["Log", dataOut] as LogEffect
}

const test = (a: void | LogEffect) => {
  a = a as LogEffect
  console.log(`test: expect main to log 1 |> ${a[0] === "Log" && a[1] === 1 }`)
} 

// The pure-function can now be tested using predictable value-producers and interceptors
pipe([() => 0.3, () => 0.7, testLog] as abc)
  .pipe(main)
  .pipe(test)
  
```

Pure Functions
==============

Pure Functions are the building blocks of any functional programming. Pure Functions take in input and return output - without mutating anything.

This therefore enforces data that is immutable.

Like Pure Actions, whenever they are passed the same set of inputs - they always return the same output.
  
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

**Pipes allow you to do the above in a much cleaner way**

In FP languages such as F# you can write the above as:

```fsharp

let a = 1 // equivalent to js const

let c = a |> increment |> increment

```

> `a` is piped into the `increment` function which is piped into the next `increment` function.

and while there is not yet (though there is a proposal for this) a pipeline operator in js / ts, Lean provides `pipe` which can be used in a similar way.

**A `pipe` is able to pipe a value through a series of functions**

```typescript

const three = pipe(1)
  .pipe(increment)
  .pipe(increment)
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

const seven = pipe(4).pipe(add(3))

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

**Javscript multi-arguments are still useful - especially when dealing with optional parameters**

```typescript

const myFunction = (a: number, b?: number) => a + ( b || 0 )

const a = myFunction(10) // 10

const a = myFunction(10, 10) // 20

```

**When dealing with many optional parameters it is also common to use objects and destructure patterns**

```typescript

const showCat = (options: { isFluffy: boolean, likesToScratch: boolean, likesIceCream: boolean }) =>
    console.log(
        `This kitty ${
            isFluffy
                ? 'is fluffy, and' 
                : ''
            } ${
                likesToScratch
                    ? ' likes to scratch, and' 
                    : ''
            } ${
                likesIceCream
                    ? ' likes ice-cream'
                    : ' that\'s all'
            }`
    )
    
showCat({ isFluffy: true }) // This kitty is fluffy, and that's all

```

**Pure Functions can also be recursive and should be used in place of loops - because loops mutate values.**

An example of a recursive function

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

pipe(myArray)
  .pipe(Array_.map(item => item + '!!!'))
  .pipe(Array_.filter(item => item !== 'vinegar!!!'))
  .pipe(Array_.reduce((ac: string, cv: string) => ac + cv)('I like '))
  .pipe(console.log)

pipe(myRecord)
  .pipe(Record.map(item => item + '!!!'))
  .pipe(Record.filter(item => item !== 'monkey!!!'))
  .pipe(Record.reduce((ac: string, cv: string) => ac + cv)('hello '))
  .pipe(console.log)

```

> Note that `Record` and `Array_` are provided in the Prelude.

Async
=====

Javascript and Typescript already provide Promises that handle asynchronicity.

`fetch` for example provides a promise as a response to reaching out and calling an outside api.

```typescript

fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then(response => response.json())
    .then(handler)

```

Infact Promises can be thought of as `pipe` for asynchronous operations.

Javascript and Typescript also provide async / await for handling asynchronicity, however this ends up introducing impurity into the code.

```typescript

const a = await fetch('https://jsonplaceholder.typicode.com/todos/1')
const b = a.json() // This introduces an impure value to b, since we can't be sure what it is.
handler(b) // handler can still be pure - but the surrounding scope here has been polluted.

```

But b never really has to come into scope, and instead `a.json()` can be passed directly to the `handler`.


```typescript

const a = await fetch('https://jsonplaceholder.typicode.com/todos/1')
handler(a.json()) // `a.json()` is passed directly to the handler and isn't an accessible value in this scope.
function handler (a_json) {
  // a_json comes into scope as a known value
}

```

Parallels
==============

Since `mutable` restricts multiple actions spawning at once, we need another type of action that is better equipped to handle the spawning of multiple actions, collecting their results and converging back to a single action.

`Promise.all` provides a standard way of doing this...

```typescript

Promise.all([
  promise1,
  promise2
]).then(handler)

```

Flows
==============

Flows are another way of running asynchronous action.
`flow` works much like `pipe` but allows both sync and async functions.

```typescript

flow(10)
  .pipe(v => v + 10)
  .pipe(logAndThrough) // logs the current value and then passes the value to the next handler.
  .pipe(v => v + 20)
  .then(wait(1000)) // This step is blocks the flow for one second
  .pipe(logAndThrough)
  
```

**Creating an Async step in the flow**

```typescript

flow(10)
  .pipe(v => v + 10)
  .then((val, resolve) => {
    // Example of how to write an async function.
    // The flow won't continue until resolve is called, and that won't happen until the timeout is reached.
    setTimeout(() => resolve(val + 90), 1000)
  }) // This step is blocks the flow for one second
  .pipe(logAndThrough)
  
```

Pattern Matching
================

In functional languages like F# most if / then / else style logic is handled through Pattern Matching.

Lean also provides pattern matching.

Here it's possible to create a type that can be matched against at run-time, and based on that match, trigger a function.

```typescript

match('hello')
  .with_($string, s => console.log(s + ' world'))
  .otherwise(_ => console.log('unable to match'))
  
```

Pattern matching takes a value and matches it against a series of patterns.
The first pattern to match, fires the value (with type inferred from the pattern) into an accompanying function.

So... let's say we have `name`.

We could do something like...

```typescript

match(name)
  .with_('garfield', matchedName => `${matchedName} is a cat`)
  .with_('odie', matchedName => `${matchedName} is a dog`)
  .otherwise(_ => console.log('unable to match'))


```

In the above `matchedName` in both cases is inferred to be a string - even though `name` may be of unknown type.
That's because `matchedName` infers it's type from the pattern.

But Pattern Matching is far more powerful than that...

### Partial Matching & Destructuring

Objects and arrays can be matched against a partial object / array.

```typescript

const a = {
  name: {
    first: 'johnny',
    last: 'bravo'
  }
}

match(a)
  .with_({ name: { first: 'johnny'} }, _ => `matching on first name`)
  .otherwise(_ => 'unable to match')


```

Which is particularly useful when used in combination with destructuring

```typescript

match(a)
  .with_({ name: { first: 'johnny'} }, ({ name: { first: b }}) => `Hey it's ${b}`)
  .otherwise(_ => 'unable to match')
```

### Runtime Interfaces

Special runtime interfaces can be used to match against in place of values...

Here we use `$string` in place of the literal 'johnny'.

```typescript

const $matchPattern = {
  name: {
    first: $string 
  }
}

match(a)
  .with_($matchedPattern, ({ name: { first: b }}) => `${b} is a string`)
  .otherwise(_ => 'unable to match')


```

It's also good to point out that a runtime interface automatically binds the correct type to the interface, so `$string` is of type `string`. So when `a` is matched, it infers the type `{ name: { first: string }}`

Runtime interfaces are powerful...

```typescript

const a = [1, 2, 3]

match(a)
  .with_($array($number), a => `${a} is an array of numbers`)
  .otherwise(_ => 'unable to match')


```

```typescript

match(a)
  .with_([1, $number, 3], ([_, b, __]) => `${b} is a number`)
  .otherwise(_ => 'unable to match')

```

```typescript

const a = {
  a: [1, 2],
  b: [3, 3, 4],
  c: [1, 5, 99]
}

match(a)
  .with_($record($array($number)), a => `A record of arrays of numbers - whoa`)
  .otherwise(_ => 'unable to match')


```

```typescript

const a = 'cat' as unknown

console.log(
  match(a)
    .with_($lt(100), _ => `< 100`),
    .with_($gt(100), _ => `> 100`),
    .with_(100, _ => `its 100`),
    .otherwise(_ => `no idea ... probably a cat`)
)

```

```typescript

const a = 'cat' as string | number

match(a)
  .with_($union([$string, $number]), _ => `a is string | number`)
  .otherwise(_ => 'unable to match')


```

Runtime interfaces include

- `$string`
- `$number`
- `$int`
- `$boolean`
- `$array([])`
- `$record()`
- `$union([])`
- `$unknown`
- `$nothing` <- Use this to match on undefined & null
- `$lt`
- `$gt`
- `$lte`
- `$gte`

## Roll your own Runtime Interfaces

```typescript

const $even =
  {
    runtimeInterface: true,
    test: (a: number) => a % 2 === 0
  } as unknown as number

const $odd =
  {
    runtimeInterface: true,
    test: (a: number) => a % 2 !== 0
  } as unknown as number

console.log(
  match(101)
    .with_($even, _ => `number is even`),
    .with_($odd, _ => `number is odd`)
    .otherwise(_ => 'unable to match')
) // number is odd

```
A Runtime interface is an object with the property `runtimeInterface: true`.
This tells the `with_` function to treat the value as a Runtime Interface.

Primitive Runtime Interfaces have a `type` property, but more complex ones have a `test` function that determines whether a match is being made.

In both `$odd` and `$even` the subject is piped into the test function and a boolean is returned which determines whether or not the subject matches.

Note that the Runtime Interface object is coerced into the expected type should the path match.

Simple, Safe Fetch
==================

```typescript

const $validJson = {
  userId: $number,
  id: $number,
  title: $string,
  completed: $boolean
}

fetch('https://jsonplaceholder.typicode.com/todos/1')
  .then(response => response.json())
  .then(json =>
    match(json)
      .with_($validJson, json => console.log(`yay - ${ json.title }`))
      .otherwise(_ => console.log(`Unexpected JSON response from API`))

  )

```

Type-cirtainty
==============

Pattern matching becomes more powerful when used to drive type-cirtainty.
The return value of pattern matching is often a `union` type or just plain `unknown`.

Instead we can drive type-cirtainty by not returning a response to a variable at all.
Instead we call a function passing in the value of cirtain-type from the inferred match.

In the below `personAction` only fires if `bob` matches `$person` so if `personAction` runs at all, then it is with type-cirtainty.

```typescript

const $person = {
  name: {
    first: $string
  }
}

type Person = typeof $person

const personAction = (person: Person) => {
  //this action runs with type cirtainty :D
  console.log(`${person.name.first} is safe`)
}

fetchPerson(123).then(
  person => match(person)
    .with_($person, personAction /* this only runs if a match occurs */)
    .otherwise(_ => console.log('not a person'))
)

```
## Conclusion

- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create complex data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables

## Prelude API

### pattern match api

**class** `Match`

**methods of `Match`**

`of`

_Creates a Match chain. Usually `match` is used instead._

Eg. `Match.of('cat')`

`with_`

```typescript

match('cat')
  .with_($string, str => console.log(`The string is ${str}`))
  .done()
  
```

_Provides a matching arm, with the thing to match against and a function to call, should the match occur..._

`done`

_Executes the patternMatch. `done` must be called at the end of the chain, in order for the matching to execute._

**function** `match`

_Used in place of `PatternMatch.of`_

**Runtime Interfaces**

- $string
- $array
- $boolean
- $gt
- $lt
- $gte
- $lte
- $literal
- $nothing
- $number
- $int
- $record
- $union
- $unknown

### pipe

**class** `Pipe`

**methods of `Pipe`** 

`of`

_Creates a Pipeable value. `pipe` is usually used instead._

`pipe`

_pipes the previous result into a new function_

Eg. `pipe(10).pipe(a => console.log(a + 10))`

`done`

_Returns the final value._

**function** `pipe`

_Used in place of `Pipe.of`_

### flow

**class** `Flow`

**methods of `Flow`**

`of`

_Creates a Flowable value. `flow` is usually used instead._

`pipe`

_pipes the previous result into a new function_

`then`

_pipes the previous result into a **potentially asynchronous** new function_

**function** `flow`

_Used in place of `Flow.of`_

#### flow helpers

**function** `wait`

_waits a period of time before passing the previous value in a flow to the next step_

Eg. `flow(10).then(wait(1000)).pipe(ten => console.log(ten))`

**function** `logAndThrough`

_Logs the previous value in a pipe or flow, and then passes that value to the next step_

### mutable

**function** `mutable`

See docs above

**function** clone

_function clone - makes a deep-clone of any array / object (dereferences the new object from the old)_

Eg. `const a = clone(b)`

### Libraries

**Array_** _See docs above_

**Record** _See docs above_
