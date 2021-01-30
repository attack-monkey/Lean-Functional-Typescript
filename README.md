# A guide to Lean Functional Typescript

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the the use of Pure Functions and Pure Actions.

Install
=======

While Lean is more conceptual than anything else, the Lean Prelude provides utilities such as **pattern matching**, **piping**, and **flows**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

`import { pipe, match } from '@attack-monkey/lean-f-ts-prelude/dist/src'`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

Lean Rules
===============

1. All values must be known before being brought into scope, or created by applying immutable operations to those known values.

For example:

```typescript

// In this example c is already in scope with a value of 10

const product_of_a_b_and_c = (a: number, b: number) => {

  // New scope begins between the curly braces. At the point at which this scope runs a, b, and c will all be known values
  
  const product_of_a_and_b = a * b // This is allowed as it has been created from known values.
  
  return product_of_a_and_b * c
}

product_of_a_b_and_c(1, 2) // 20

```

2. Once a variable has been brought into scope, it must not be mutated. To enforce this, mutability is handled by `mutatble`, which abides by this rule.

```typescript

const [ unwrap_a, mutate_a ] = mutable(10)

unwrap_a(a => {
  
  console.log(`a is 10: ${a}`)
  
  mutate_a(a => a + 10) // While this mutates `a` to 20, the new state is only accessible by (re)unwrapping the new state.
  
  console.log(`a is still 10 in this scope ${a}`)
  
  unwrap_a(a => {
    console.log(`In this new scope, a is 20: ${a}`)
  })
  
})

```

By following these two rules, any 'do operations' are abstracted away from the otherwise Pure Program. No 'do operations' cause side-effects in the currently running code.

These two guiding Rules form the basis of Lean Functional Typescript.

Actions
======

In Lean, functions that 'do things' other than just return a result, are referred to as Action. Actions that abide by the above rules, and otherwise behave as Pure Functions, are known as Pure Actions.

A note on `undefined`, `void`, and `null`
================================

In functional languages, a pure function must return a value - even if it's an empty value - usually referred to as unit.

In javascript, typescript, and Lean, a function is allowed to return `undefined`, `null`, and `void`.
Note that in typescript, when a function doesn't return a value at all (`void`), this implicitly returns `undefined`.

Eg.

```typescript

type MyFn = () => void

const myFn: MyFn = () => {}

console.log(myFn()) // undefined

```

In Lean a function can still return `undefined`, `void`, `null`, and still be regarded as pure!

Pure Macros
===========

The `now` Action below, is an example of how a Pure Action can be written to wrap around it's impure variant.

```typescript

type Now = <A>(handler: (n: number) => A) => A

const now: Now = handler => handler(Date.now())

const handler = (t: number) => console.log('the time is ' + t)

now(handler)
```

When `now` gets called it passes the result into the `handler`. 
The `handler` is pure as it always returns the same thing - undefined. 
If we were to log `now(handler)`, we would see that the return value is also always `undefined`, and therefore pure.

The trick is that rather than leak impurity into any currently running function / macro, Pure Actions call an instance of the Handler Action - passing in the impure result as an input. Nothing that is currently running is affected by the operation.

### Promises

Promises already conform to the notion of Pure Action, and infact by wrapping `Date.now()` in `Promise.resolve` we end up with a very similar result to above...

```typescript

Promise.resolve(Date.now()).then(handler)

```

Wrapping Impurities
===================

Pure Actions that wrap Impure Actions can be thought of as an integration to the Impure World outside of our otherwise Pure Program. 
Writing Pure Actions to wrap your own Impure code - should be avoided, and instead Impure code should be re-written to be Pure.
The only impurities that should be wrapped are native impurities and third-party impurities.

Immutable Operations
====================

In Lean, **once a variable has been brought into scope - it cannot be changed**. In other words all variables should be considered immutable.

So instead of mutating variables, use immutable operations and pure functions, and assign the result to a new variable.

Eg.

```typescript

const a = 'hello'
const b = a + ' world'

console.log(a) // hello (i.e. no mutation)
console.log(b) // hello world

```

or pipe the result of one function into the next function...

```typescript

const appendString = 
    (str: string) =>
        (originalStr: string) =>
            originalStr + str
        
pipe('hello')
    .pipe(appendString(' world'))
    .pipe(console.log)

```

Mutables
========

Mutability can however be managed in a completely pure way - by abiding by the Rules of Lean.

`mutable` returns a tuple containing unwrap and mutate macros.

Calling the mutate macro calls the handler passing in the current value. The return value of the handler becomes the new value, however nothing in any currently running macro is mutated.

When the upwrap macro is called it's handler is called, passing in the now updated value.

```typescript

const [unwrapCat, mutateCat] = mutable('garfield')

mutateCat(_ => 'felix')

mutateCat(cv => cv + '!')

unwrapCat(console.log) // felix!

```

Calling `mutateCat` does not mutate any value in any currently running macro, and can therefore be considered a Pure Action.

**Using a Mutable Tuple to organise parallel operations**

```typescript

type TempTuple = [string | undefined, string | undefined, string | undefined]
const [unwrapTuple, mutateTuple] = mutable([ undefined, undefined, undefined ] as TempTuple)

const runIfComplete = (tup: TempTuple) => {
    if (tup.every(item => !!item)) {
        console.log(tup)
    }
}

mutateTuple(tup => ['purple', tup[1], tup[2]])
unwrapTuple(runIfComplete)
mutateTuple(tup => [tup[0], 'monkey', tup[2]])
unwrapTuple(runIfComplete)
mutateTuple(tup => [tup[0], tup[1], 'dishwasher'])
unwrapTuple(runIfComplete) // ["purple", "monkey", "dishwasher"]

```

**An incremental id generator using mutables**

```typescript

type NewIdGen = () => (f: (cv: number) => void) => void

const newIdGen: NewIdGen = () => {
    const [unwrapId, mutateId] = mutable(0)
    return f => {
        mutateId(id => id + 1)
        unwrapId(id => f(id))
    }
}

const myIdGen = newIdGen()
myIdGen(id => console.log(id)) // 1
myIdGen(id => console.log(id)) // 2

```

**Creating a loop where by a mutation event is followed by recalling the unwrap event until a condition is met.**

```typescript

const [unwrapValue, mutateValue] = mutable(10)

const loop = () => unwrapValue(a => {
    console.log(`val before: ${a}`)
    // This mutation event does not mutate the current macro - so logging before and after this line will yield the same value.
    mutateValue(a => a + 1)
    if (a < 12) loop()
    console.log(`val after: ${a}`)
})

loop()

```

When an unwrap or mutate macro is called, the handler recieves the 'unwrapped value', and the handler itself represents the 'unwrapped context'.
In this 'unwrapped context' - the unwrapped value cannot be mutated directly. Calling the mutate macro only mutates the state of the mutable, but that new state is only made available upon creating a new 'unwrapped context'. This ensures that `mutable` doesn't violate purity by mutating values in a currently running macro.

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

Since `mutable` restricts multiple macros spawning at once, we need another type of macro that is better equipped to handle the spawning of multiple macros, collecting their results and converging back to a single macro.

`Promise.all` provides a standard way of doing this...

```typescript

Promise.all([
  promise1,
  promise2
]).then(handler)

```

Flows
==============

Flows are another way of running asynchronous macros.
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
  .with_($unknown, _ => console.log('unable to match'))
  .done()
  
```

Pattern matching takes a value and matches it against a series of patterns.
The first pattern to match, fires the value (with type inferred from the pattern) into an accompanying function.

So... let's say we have `name`.

We could do something like...

```typescript

match(name)
  .with_('garfield', matchedName => `${matchedName} is a cat`)
  .with_('odie', matchedName => `${matchedName} is a dog`)


```

In the above `matchedName` in both cases is inferred to be a string - even though `name` may be of unknown type.
That's because `matchedName` infers it's type from the pattern.

Pattern Matching can be used to return a value.
The result is the result of the function that fires upon match.
If there is no match, then the original value is returned instead.
To return a value, end the chain in `.done()`.


```typescript

const name: string = getName()

const a = match(name)
  .with_('garfield', matchedName => `${matchedName} is a cat`)
  .with_('odie', matchedName => `${matchedName} is a dog`)
  .done()

```

In the above, since the value and both `with_` arms all return a string - the compiler is smart enough to know that the resulting type is always string. Therefore `a` gets an inferred type of string.

If one of the arms returned a `number` then `a` would have an inferred type of `string | number`.

### Literal matching

We've already seen how simple equality matches can be made...

```typescript

const a = 'cat' as unknown

const b = match(a)
  .with_('cat', _ => `hello kitty`),
  .with_('dog', _ => `hello doggy`)


```

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
  .with_({ name: { first: 'johnny '} }, _ => `matching on first name`)


```

Which is particularly useful when used in combination with destructuring

```typescript

match(a)
  .with_({ name: { first: 'johnny '} }, ({ name: { first: b }}) => `Hey it's ${b}`)

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


```

It's also good to point out that a runtime interface automatically binds the correct type to the interface, so `$string` is of type `string`. So when `a` is matched, it infers the type `{ name: { first: string }}`

Runtime interfaces are powerful...

```typescript

const a = [1, 2, 3]

match(a)
  .with_($array($number), a => `${a} is an array of numbers`)


```

```typescript

match(a)
  .with_([1, $number, 3], ([_, b, __]) => `${b} is a number`)

```

```typescript

const a = {
  a: [1, 2],
  b: [3, 3, 4],
  c: [1, 5, 99]
}

match(a)
  .with_($record($array($number)), a => `A record of arrays of numbers - whoa`)


```

```typescript

const a = 'cat' as unknown

console.log(
  match(a)
    .with_($lt(100), _ => `< 100`),
    .with_($gt(100), _ => `> 100`),
    .with_(100, _ => `its 100`),
    .with_($unknown, _ => `no idea ... probably a cat`) // Use $unknown as a catch all
    .done()
)

```

```typescript

const a = 'cat' as string | number

match(a)
  .with_($union([$string, $number]), _ => `a is string | number`)


```

Runtime interfaces include

- `$string`
- `$number`
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
    .done()
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
      .with_($unknown, a => console.log(`Unexpected JSON response from API`))
      .done()
  )

```

Type-cirtainty
==============

Pattern matching becomes more powerful when used to drive type-cirtainty.
The return value of pattern matching is often a `union` type or just plain `unknown`.

Instead we can drive type-cirtainty by not returning a response to a variable at all.
Instead we call a macro passing in the value of cirtain-type from the inferred match.

In the below `personAction` only fires if `bob` matches `$person` so if `personAction` runs at all, then it is with type-cirtainty.

```typescript

const $person = {
  name: {
    first: $string
  }
}

type Person = typeof $person

const personAction = (person: Person) => {
  //this macro runs with type cirtainty :D
  console.log(`${person.name.first} is safe`)
}

fetchPerson(123).then(
  person => match(person)
    .with_($person, personAction /* this only runs if a match occurs */)
    .with_($nothing, _ => console.log('not a person'))
    .done()
)

```
## Conclusion

- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create complex data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables
- Pure Actions 'do something' as well as return a value. When dealing with impure values, Pure Actions call new instances of Child Actions (AKA Handlers) and pass impure values as inputs. They do this instead of returning an impure value inside a pure function which would otherwise pollute the pure function.
- By combining Pure Actions and Pattern Matching, Type Certainty can be achieved making code extremely predictable and safe.

## Prelude API

### pattern match api

**class** `PatternMatch`

**methods of `PatternMatch`**

`of`

_Creates a PatternMatch chain. Usually `match` is used instead._

Eg. `PatternMatch.of('cat')`

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
