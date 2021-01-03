# A guide to Lean Functional Typescript

Lean is a Pure Functional way of writing Typescript applications.

It achieves purity through the concept of Pure Functions and Pure Macros.

Install
=======

While Lean is more conceptual than anything else, the Lean Prelude provides utilities such as **pattern matching**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.

Lean Principles
===============

1. **Every impurity can be wrapped in a 'Pure' Macro.** (Note: Any function that does something other than just return a result, is known as a macro in Lean. Functions calculate things. Macros do things).

2. **Once a variable has been declared - it cannot be changed**

3. **Values / Data should not be polluted with methods. Instead functions and Macros accept Values of a particular 'Call Signature'**.

These three guiding principles form the basis of Lean Functional Typescript.

Pure Macros
===========

- When passed the same set of inputs, Pure Macros always return the same result (even if that result is undefined), and therefore appear pure to the currently running function / macro.
- Pure Macros do not mutate **anything** in any running functions / macros.

In this way `console.log` is a pure macro but `Date.now()` is not.
`Date.now()` is considered impure as it returns a different value each time it is called.

Rather than return impure values, Pure Macros take a 'Handler' as an argument. The Handler is itself a PureMacro, designed to accept the impure result as it's argument.

The `now` macro below, is an example of how a Pure Macro can be written to wrap around it's impure variant.

```typescript

const now = <A>(f: (n: number) => A) => {
    f(Date.now())
}

const handler = (t: number) => console.log('the time is ' + t)

now(handler)

```

When `now` gets called it passes the result into the `handler`. 
The `handler` is pure as it always returns the same thing - undefined. 
If we were to log `now(handler)`, we would see that the return value is also undefined, and therefore pure.

The trick is that rather than leak impurity into any currently running function / macro, Pure Macros spawn an instance of the Handler Macro - passing in the impure result as an input. Nothing that is currently running is affected by the operation.

### Promises

Promises already conform to the notion of Pure Macro, and infact by wrapping `Date.now()` in `Promise.resolve` we end up with a very similar result to above...

```typescript

Promise.resolve(Date.now()).then(handler)

```

Immutable Operations
====================

In Lean, **once a variable has been declared - it cannot be changed**. In other words all variables should be considered immutable.

So instead of mutating variables, use immutable operations and pure functions, and assign the result to a new variable.

Eg.

```typescript

const a = { greeting: 'hello ', thing: '' }
const b = { ...a, thing: 'world' }

console.log(a) // hello (i.e. no mutation)
console.log(b) // hello world

```

or pipe the result of one function into the next function...

```typescript

const changeThing = 
    <A>(newThing: A) => 
        (obj: { thing: A }) =>
            ({ ...obj, thing: newThing })

pipe({ greeting: 'hello ', thing: '' })
    .pipe(changeThing)
    .pipe(console.log)

```

Mutables
========

Mutability can however be managed in a completely pure way - by abiding by the rules of Pure Macros.
`pureMutable` *available from Lean Prelude* takes a starting value along with a 'Handler'.
It spawns a new instance of the Handler passing in the initial value as input, as well as a special `fset` (functional set) macro.
When `fset` is called, nothing is mutated in any currently running macros, BUT a new Handler Macro is spawned, passing in the new value (`currentValue`) as input.
Actually both the old and new value are passed into the Handler should it need to work with the old and new.
Both `PureMutable` and `fset` appear pure to the functions / macros calling it.

_We'll get to the `pureSetTimeout` in a moment..._

```typescript

pureMutable(1)(({ currentValue, fset }) => {
    pureSetTimeout(1000)(() => {
        console.log(currentValue)
        fset(currentValue + 1)
    })
})

```

Note that PureMutable also comes with some safe-guards. 
1. `fset` can only be called if a time buffer is in place between running the `pureMutable` and calling `fset`. This prevents inifnite looping, and also prevents mutation within the Handler Macro.
2. `fset` is only callable once per running instance of the Handler. This stops multiple instances running at the same time and forces a safer way of working with data.

```typescript

pureMutable(1)(({ currentValue, fset }) => {
    fset(currentValue + 1) // This will cause an error because fset cannot be called synchronously
    pureSetTimeout(1000)(() => {
        console.log(currentValue)
        fset(currentValue + 1) // This fset is ok because there is a time buffer between the pureMutable and the fset.
        fset(currentValue + 2) // This fset is not ok, because the previous fset would have already been triggered.
    })
})

```

The `pureSetTimeout` above is a pure version of `setTimeout` and is defined as 

```typescript

// A Pure Wrapper around setTimeout
const pureSetTimeout =
    (timeout: number) =>
        <A>(f: () => A) => {
            setTimeout(f, timeout)
        }
 
 ```

Another example of using `pureSetTimeout` with `pureMutable`...

```typescript

// if value is < 10, first beep and then set a new value of n + 1
const handler =
    (value: number) =>
        (fset: (a: number) => void) =>
            () => {
                if (value < 10) {
                    console.log(`beep ${value}`)
                    fset(value + 1)
                }
            }

// Set up a PureMutable that holds a number
pureMutable(1)(
    ({ currentValue, fset }) =>
        // On first run, and then whenever the value changes - call the PureSetTimeout function
        pureSetTimeout
            (1000)  // The PureSetTimeout will wait 1 second before calling the handler
            (
                handler // The handler takes a value and a fset function 
                    (currentValue)
                    (fset)
            )
)

```


Working with PureMutable can take some time to get used to. It's best to think of it as a way to hold multiple variables at a particular scope. A global scope `pureMutable` can exist outside of the main app function. When the global scope changes - it spawns a new instance of the main app function.

Layers of lower level scoped `pureMutable`'s can then be used in smaller components.

### Other useful PureMutable helpers

The `PureMutable` Handler accepts an object with the following keys that we have already seen:

- currentValue
- fset

But also:

- previousValue
- fsetThenIgnore

`previousValue` is pretty self explanitory and as expected returns the previous value (before the fset).

`fsetThenIgnore` is similar to `fset` however it will not throw an error in the event that multiple `fset` calls occur from the same macro. It still only allows the first operation to occur - but silently rejects any thereafter. This is useful when race conditions are tolerated.

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

### Pipes

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

**When dealing with many optional parameters it is also common to use objects and destructure patterns, as is seen with pureMutable**

```typescript

pureMutable(1)(
    ({ currentValue, fset }) => { ... }
)

```

**Pure Functions can also be recursive and should be used in place of loops - because loops mutate values.**

```

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

It is common to see functional libraries provide classes that 'lift' values to provide methods that then work on the value or collection of values inside.

```typescript

const myList = List.of(1, 2, 3)

const myNewList = myList
  .reverse()
  .append('blast off!')
  .done()

```

> Note: In essence this is what the `pipe` function itself does, however Lean uses `pipe` as a utility filling the gap of not having a pipeline operator.

So while 'lifting' a value into a context that provides the value with methods is a legit way of performing functional programming in javascript... it is not 'Lean'. It means that `reverse` and `append` are bound to the `List` class. If those methods are then to be used in another class, we have to either write those methods again, or use class extensions, or messy mixins.

If however `reverse` and `append` are simply stand alone functions, then we can use pipes to connect the functions together.

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
const b = a.json() // This introduces an impurure value into the code.
handler(b) // handler can still be pure - but this function / macro has been polluted.

```

So `promise` syntax is preferred in Lean.

Parallels
==============

Since `PureMutable` restricts multiple macros spawning at once, we need another type of macro that is better equipped to handle the spawning of multiple macros, collecting their results and converging back to a single macro.

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
  with_($matchedPattern, ({ name: { first: b }}) => `${b} is a string`)


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
- Pure Macros 'do something' as well as return a value. When dealing with impure values, Pure Macros spawn new instances of Child Macros and pass impure values as inputs. They do this instead of returning an impure value inside a pure function which would otherwise pollute the otherwise pure function.

