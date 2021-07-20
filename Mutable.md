the `mutable` function which is a safer utility for managing mutability. 

It ensures that a value that has been brought into scope does not change unexpectedly. 

It does so by abstracting away the persistance of a value from the otherwise Pure Program.

`mutable` is a part of the Lean Prelude.

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
