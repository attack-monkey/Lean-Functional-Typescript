# A guide to Lean Functional Typescript

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the concept of Pure Functions and Pure Macros.

Install
=======

While Lean is more conceptual than anything else, the Lean Prelude provides utilities such as **pattern matching**, **piping**, and **flows**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

Lean Principles
===============

1. **Every impurity can be wrapped in a 'Pure' Macro' / 'Pure Function'.** (Note: Any function that does something other than just return a result, is known as a macro in Lean. Functions calculate things. Macros do things).

2. **Once a variable has been declared - it cannot be changed**

These two guiding principles form the basis of Lean Functional Typescript.

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

- A Macro is a function that 'does' something other than just return a result. 
- All Macros in Lean are either pure OR native / third-party impurities, that are wrapped by a Pure Macros.
- Pure Macros abstract the 'doing' part (aka side-effect) out of the program and otherwise behave as pure functions.
- When passed the same set of inputs, Pure Macros always return the same result (even if that result is undefined), and therefore appear as pure functions to the currently running function / macro.
- Pure Macros do not mutate **anything** in any running functions / macros.

In this way `console.log` is a pure macro but `Date.now()` is not.
`Date.now()` is considered impure as it returns a different value each time it is called.

Rather than return impure values, Pure Macros take a 'Handler' as an argument. The Handler is itself a PureMacro, designed to accept the impure result as it's argument.

The `now` macro below, is an example of how a Pure Macro can be written to wrap around it's impure variant.

```typescript

type Now = <A>(handler: (n: number) => A) => A

const now: Now = handler => handler(Date.now())

const handler = (t: number) => console.log('the time is ' + t)

now(handler)
```

When `now` gets called it passes the result into the `handler`. 
The `handler` is pure as it always returns the same thing - undefined. 
If we were to log `now(handler)`, we would see that the return value is also always `undefined`, and therefore pure.

The trick is that rather than leak impurity into any currently running function / macro, Pure Macros call an instance of the Handler Macro - passing in the impure result as an input. Nothing that is currently running is affected by the operation.

### Promises

Promises already conform to the notion of Pure Macro, and infact by wrapping `Date.now()` in `Promise.resolve` we end up with a very similar result to above...

```typescript

Promise.resolve(Date.now()).then(handler)

```

Wrapping Impurities
===================

Pure Macros that wrap Impure Macros can be thought of as an integration to the Impure World outside of our otherwise Pure Program. 
Writing Pure Macros to wrap your own Impure code - should be avoided, and instead Impure code should be re-written to be Pure.
The only impurities that should be wrapped are native impurities and third-party impurities.

Block Scoping Impurities
========================

Another acceptable way of purifying native and third-party impurities, such as Date.now and Math.random, is to block-scope
the assignment of the impure values. In this way, the impure values cannot leak beyond the boundaries of the block, and the only way out is to call a macro.
Providing that this macro is pure, then the block-scope can be considered pure.

```typescript

{
  const d1 = Date.now()
  const d2 = Math.random()
  out(d1, d2)
}
function out(d1:number, d2: number) {
  console.log(d1, d2)
}

```

A Note on partial functions...
==============================

Partial Functions occur when a function returns a function that then uses both the outer and inner scope to form a result. Technically the returned function is using scope outside of it's direct inputs to calculate a result and could be considered impure. However, since there is no way of calling the inner function other than by calling the outer function, everything is considered safe and pure. Partial functions are also referred to as curried functions, and are a common part of functional programming in general.

```typescript

const fn1 = (a: number) => (b: number) => a + b  

```

### Taking Partials Further...

Values in an outer scope can be used safely - so long as macros obey the rule of not mutating the state of any running macro.
With that rule being followed, then functions and macros that use values from an outer-scope will still return the same value when passed a given set of inputs - at least in the macro / context they are running in (Since values (including outer-scope values) cannot change during the running of any macro).

This is effectively an extension on the concept of Partial Functions.

These types of functions / macros cannot be called from outside of the function / macro they are scoped to.
These types of functions / macros MUST all eventually roll up to a Pure Function / Macro otherwise the application fails to be Pure.

```typescript

const myPureMacro = (): void => {
    now(n1 => {
        const handler =
            (n2: number) => n1 + n2 // `n1` will always be the same throughout the running of myPureMacro.
            
        console.log(
            now(handler)
        )
        
    })
}

```

Immutable Operations
====================

In Lean, **once a variable has been declared - it cannot be changed**. In other words all variables should be considered immutable.

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

Mutability can however be managed in a completely pure way - by abiding by the rules of Pure Macros.

`mutable` returns a tuple containing unwrap and mutate macros.

Calling the mutate macro calls the handler passing in the current value. The return value of the handler becomes the new value, however nothing in any currently running macro is mutated.

When the upwrap macro is called it's handler is called, passing in the now updated value.

```typescript

const [unwrapCat, mutateCat] = mutable('garfield')

mutateCat(_ => 'felix')

mutateCat(cv => cv + '!')

unwrapCat(console.log) // felix!

```

Calling `mutateCat` does not mutate any value in any currently running macro, and can therefore be considered a Pure Macro.

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

const id1 = newIdGen()
id1(id => console.log(id)) // 1
id1(id => console.log(id)) // 2

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

When the unwrap or mutate macro is called, the handler recieves the 'unwrapped value' and the handler itself represents the 'unwrapped context'.
In this 'unwrapped context' - the unwrapped value DOES NOT MUTATE. Calling the mutate macro only mutates the state of the mutable, but that new state is only made available upon calling unwrap or mutate again. This ensures that `mutable` doesn't violate purity by mutating values in a currently running macro.

Pure Functions
==============

Pure Functions are the building blocks of any functional programming. Pure Functions take in input and return output - without mutating anything.

This therefore enforces data that is immutable.

Like Pure Macros, whenever they are passed the same set of inputs - they always return the same output.
  
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

**Which are easier to use in flows than regular multi-argument functions**

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

It is common to see functional libraries provide classes that 'lift' values to provide methods that then work on the value or collection of values inside...

```typescript

const myList = List.of(1, 2, 3)

const myNewList = myList
  .reverse()
  .append('blast off!')
  .done()

```

So while 'lifting' a value into a context that provides the value with methods is a legit way of performing functional programming in javascript... it is not preferred in Lean. In the above it would mean that `reverse` and `append` are bound to the `List` class. If those methods are then to be used in another class, we have to either write those methods again, or use class extensions, or messy mixins.

> Note: In essence, the `pipe` function is a wrapper around a `Pipe` class that lifts a value into a 'pipeable context'. That is, it binds the `pipe` and `done` methods to `this`. However Lean uses `pipe` as a utility for moving data from one function to the next, rather than transforming data. Therefore the number of methods on `pipe` is minimal, and doesn't suffer from the need to extend the class, etc. The same goes for native Promise syntax, and even Rxjs.

So, in the above, if `reverse` and `append` are simply stand alone functions, then we can use pipes to connect the functions together.

```typescript

const myNewList =
  pipe([1,2,3])
    .pipe(reverse)
    .pipe(append('blast off!'))
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

Even though the `Array_` and `Record` library have a set of methods, the data being passed into these methods are not bound to those methods.
For example, there may be a different library that acts on just number arrays - let's call it `NumberArray`. An array of numbers would therefore be able to be passed into any of the methods from both `Array_` and `NumberArray` and may look something like...

```typescript

pipe([1, 2, 3])
  .pipe(Array_.map(item => item * 2))
  .pipe(NumberArray.sum)
  .pipe(console.log)

```

## If you do need to write functional classes... you can do so without mutating properties...

It's possible to write classes that don't mutate properties and that still achieve chainability.

In this example, rather than mutate the property `a`, the `pipe` method returns a new `Pipe` object altogether.

This is essentially the `pipe` function that the Lean Prelude provides...

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
handler(b) // handler can still be pure - but this function / macro has been polluted.

```

So `promise` syntax is preferred in Lean.

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

Pattern Matching - In Depth
===========================

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
      .with_($validJson, json => console.log(`yay - ${ json.title }`)),
      .with_($unknown, a => console.log(`Unexpected JSON response from API`))
  )

```

Type-cirtainty
==============

Pattern matching becomes more powerful when used to drive type-cirtainty.
The return value of pattern matching is often a `union` type or just plain `unknown`.

Instead we can drive type-cirtainty by not returning a response to a variable at all.
Instead we call a macro passing in the value of cirtain-type from the inferred match.

In the below `personMacro` only fires if `bob` matches `$person` so if `personMacro` runs at all, then it is with type-cirtainty.

```typescript

const $person = {
  name: {
    first: $string
  }
}

type Person = typeof $person

const personMacro = (person: Person) => {
  //this macro runs with type cirtainty :D
  console.log(`${person.name.first} is safe`)
}

fetchPerson(123).then(
  person => match(person)
    .with_($person, personMacro /* this only runs if a match occurs */),
    .with_($nothing, _ => console.log('not a person'))
)

```
## Conclusion

- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create complex data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables
- Pure Macros 'do something' as well as return a value. When dealing with impure values, Pure Macros call new instances of Child Macros (AKA Handlers) and pass impure values as inputs. They do this instead of returning an impure value inside a pure function which would otherwise pollute the pure function.
- By combining Pure Macros and Pattern Matching, Type Certainty can be achieved making code extremely predictable and safe.

