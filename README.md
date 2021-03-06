# A guide to Lean Functional Typescript

Lean is a Functional way of writing Typescript applications, based on the ML family of languages.

It provides a **lean** way of writing safer code without the complexity of some other functional languages.

It differs from say fp.ts which has aligns many haskell-like features to Typescript, but at the cost of complexity.

Install
=======

The Lean Prelude provides utilities such as **pattern matching** and **piping**.

To get going with Lean, download the **prelude**

`npm i @attack-monkey/lean-f-ts-prelude`

`import { pipe, match } from '@attack-monkey/lean-f-ts-prelude/dist/src'`

It works with both javascript and typescript...

We recommend typescript for the type-safety that it gives.


Pure Functions
==============

Pure Functions are the building blocks of any functional programming. Pure Functions take in input and return output - without mutating anything.

This therefore enforces data that is immutable.

Whenever they are passed the same set of inputs - they always return the same output.
  
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

Type Lifting
============

"Type lifting" is when a normal value without methods is 'lifted' to a more powerful version that has methods associated with it.

In Javascript automatic Type Lifting occurs for most value types, for example:

```typescript

// 10 is a value with type of number
10

// 10 is also lifted to being an instance of Number which grants it methods.
(10).toString()

// 10 can also utilise the Number static methods...
Number.isInteger(10) // true
// or
pipe(10)
  .pipe(Number.isInteger)
  .done()

```

Automatic Type Lifting also occurs on arrays...

```typescript
// [1,2,3] is an array of numbers
[1,2,3]


// [1,2,3] is also lifted to being an instance of Array which grants it methods.
[1,2,3]
  .map(x => x +1)
  .filter(x => x > 1)

```

In Lean we sparingly use Type Lifting, and tend to use piping for most things instead. This lets functions / static methods do the heavy lifting, rather than having to write classes representing Lifted Types that contain methods that are only usable in that given class.

In saying that, `pipe` is a lifted type that grants values the `pipe` method - so obviously they can be very useful ;)

And as you can see above, `[1,2,3].map(x => x +1).filter(x => x > 1)` is nice and succinct and still functional.

Mutables
========

In Js / Ts there is (unfortunately) no shortage of ways to write mutable code.
However, in functional code - mutating code is used very sparingly, and instead immutable patterns (like Pure-functions) are greatly encouraged.

Lean provides the `mutable` function which is a safer utility for managing mutability. It ensures that a value that has been brought into scope does not change unexpectedly. It does so by abstracting away the persistance of a value from the otherwise Pure Program.

```typescript

const [unwrapThing, mutateThing] = mutable(100)

unwrapThing(v => {
  // v is 100
  console.log(v) // 100
  mutateThing(m => m + 1)
  console.log(v) // The currently unwrapped value is still 100 so that there is no change to values in the existing scope.
  // To get the newly mutated value - the value needs to be explicitly unwrapped again.
  unwrapThing(v => {
    console.log(v) // 101
  })
})

```

If you do have to mutate a value in a normal Js / Ts way and that value is an object / array - then take caution:

Object properties are just references to underlying values, and when you edit an object - you actually update the underlyng value that the Object property points to. That means that when you copy an Object, you just create a reference Object to the same underlying values. Update a property on one Object and you've just updated the same property on the other Object.

Lean provides the `clone` utility that deep clones an Object to create a completely new Object.

```typescript

const a = {
  prop1: "hello world"
}

const b = clone(a)
b.prop1 = "goodbye world"
console.log(a.prop1) // "hello world" <- Yay, we didn't mutate a when mutating b

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

`Promise.all` provides a standard way of processing multiple promises at once...

```typescript

Promise.all([
  promise1,
  promise2
]).then(handler)

```

Pattern Matching
================

In functional languages like F# most if / then / else style logic is handled through Pattern Matching.

Lean provides powerful pattern matching, both with structural matches and matching on variants.

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
- `$encoded` <- Use this to match on variants - eg. Option, Some, None

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

Matching using Variants
=======================

Variants are a way of encoding values that may be one of many types. Unlike the structural pattern matching used above, variants provide a light-weight way of passing around uncertainty, and then being able to match and unwrap a definite type later.

To create a variant...

```typescript

// Create base types that encode values
type Some<A> = { t: "Some", v: A }
type None = {t: "None", v: undefined }

// Create the variant type
type Option<A> = Some<A> | None

// Create constructors that return the variant type
const Some = <A>(a: A): Option<A> => ({
    t: "Some",
    v: a
})

const None = (a: any): Option<any> => ({
    t: "None",
    v: undefined
})

// Values can now be encoded as a variant
const cat = Some("cat")

// Now the variant can be handled by switching on the type (t).
// Each case knows that since the type (t) is defined, the type of the value (v) is known. 
switch (cat.t) {
    case "Some": console.log(`hello ${cat.v}!!`); break;
    case "None": console.log(":("); break;
    default: console.log(":("); break;
}

```

Conclusion
==========

- Lean is heavily based on the ML family of languages but is idiomatic to Js / Ts.
- Lean focusses on data.
- Data has a particular type, and in order to be passed into a function, the type needs to match the call signature of the function.
- Data is piped through functions to create data transformations.
- Pure Functions take in data, and return new data without mutating the input or any other variables.
- Pattern Matching is used in place of if / else / switch for more powerful and simpler logic handling.

Prelude API
===========

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
- $encoded

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

### mutable

**function** `mutable`

See docs above

### Js / Ts Helpers

**function** clone

_function clone - makes a deep-clone of any array / object (dereferences the new object from the old)_

Eg. `const a = clone(b)`

### Libraries

**Array_** _See docs above_

**Record** _See docs above_
