# I ❤ Ramda - Beyond Map, Filter, and Reduce

[![license CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg)](https://creativecommons.org/licenses/by/4.0/)

![chemicals in flasks](chemicals_in_flasks-cropped.jpg)

https://commons.wikimedia.org/wiki/File:Chemicals_in_flasks.jpg

Congratulations, you have mastered `map`, `filter`, and `reduce`! That is no small feat, but `map`, `filter`, and `reduce` is only the beginning of the journey. Let's continue to build on this knowledge and see what we can do when we go beyond `map`, `filter`, and `reduce` with Ramda.

# The Familiar

Let's start with an example you are already familiar with.

```javascript
const values = [1, 2, 3, 4, 5];
const result = values.map(x => x * 2);
// => [2, 4, 6, 8, 10]
```

This code is a great starting point. Let's see how it can be improved.

# The Breakdown

If you take a close look at the code above, you might have noticed a teeny-tiny anonymous function.

```javascript
const result = values.map(x => x * 2);
//                        ----------
//       This function... ^
```

This function does't seem like it's out of place, but it should be extracted out of `map`. It's hard to see _why_ in this example, because `x => x * 2` is so very small and easy to read. But this is not what you'll encounter in the real world, so let's imagine a more realistic example.

```javascript
// BAD: What is this sorcery?
values.map(r => (4 / 3) * Math.PI * Math.pow(r, 3));

// GOOD: Very clear!
const radiusToAreaOfSphere = r => (4 / 3) * Math.PI * Math.pow(r, 3);
values.map(radiusToAreaOfSphere);
```

The BAD example requires you to read through the function to discover it's purpose. **You should never have to read the contents of a function to learn what a function does**. The GOOD example doesn't require you to even read the formula. You can stop reading at the name as the intent is clearly stated. Plus that whole reusability thing.

Now let's do the same thing to double.

```javascript
// double is extracted
const double = x => x * 2;
const values = [1, 2, 3, 4, 5];
const result = values.map(double);
// => [2, 4, 6, 8, 10]
```

Next I'd like to focus on the list transformation `values.map(double)`. This can be made a standalone function for the same reasons, reusability and readability.

```javascript
// create doubleAll standalone function
const double = x => x * 2;
const doubleAll = values => values.map(double);
const values = [1, 2, 3, 4, 5];
const result = doubleAll(values);
// => [2, 4, 6, 8, 10]
```

Now `double` and `doubleAll` can be moved into an external library.

```javascript
// lib/mathHelpers.js
export const double = x => x * 2;
export const doubleAll = values => values.map(double);
```

Now the code looks like this:

```javascript
import { double, doubleAll } from "./lib/mathHelpers";

const values = [1, 2, 3, 4, 5];
const result = doubleAll(values);
// => [2, 4, 6, 8, 10]
```

# Enter Ramda

With a small tweak we can easily modify our `doubleAll` function to use Ramda's `map` instead of `Array`'s `map`.

```javascript
// Array's map
export const doubleAll = values => values.map(double)

// Ramda's map
export const doubleAll = map(double)
```

While `Array`'s map only works on an `Array`, Ramda's `map` will also work on an `Object` and a `String`. So we can also do things like this:

```javascript
const charCodeAt = x => x.charCodeAt(0);
const stringToCharCodes = map(charCodeAt);

stringToCharCodes("ABC");
// => [65, 66, 67]
```

or this...

```javascript
const times100 = x => x * 100;
const times100All = map(times100);

times100All({ a: 1, b: 2, c: 3 });
// => { a: 100, b: 200, c: 300 }
```

Now that is bad ass!

# Mapping over objects

Now before you you go asking, "but when would I ever want to map over an `Object`?", let me show you some real world use cases.

I can easily create a function to `promisify` all functions in an `Object`.

```javascript
/**
 * lib/promisifyAll.js
 */
import map from "ramda/src/map";
import when from "ramda/src/when";
import is from "ramda/src/is";
import { promisify } from "util";

export const promisifyAll = map(when(is(Function), promisify));
```

Now I can `promisify` all the things!

```javascript
import { promisifyAll } from "./lib/promisifyAll";
import fs from "fs";

const fsp = promisifyAll(fs);

fsp.readFile("package.json", "UTF8").then(file => console.log(file))(
  // or we could use async/await!
  async () => {
    const file = await fsp.readFile("package.json", "UTF8");
    console.log(file);
  }
)();
```

`curry` all functions in an `Object`...

```javascript
import curryAll from "./lib/curryAll";

// some pretend math object
const math = curryAll({
  add: (a, b) => a + b,
  multiply: (a, b) => a * b
});

const increase = math.add(1);
const double = math.multiply(2);

increase(3); // => 4
double(7); // => 14
```

**Extra Credit**

Since these functions seem to be doing the same thing and I want to be [DRY](https://github.com/ryanmcdermott/clean-code-javascript#remove-duplicate-code), I'll extract out the common functionality to a new function.

```javascript
/**
 * decorates all functions in an object with func.
 */
const decorateAll = func => map(when(is(Function), func));
```

Now we can create the above functions like this:

```javascript
const curryAll = decorateAll(curry);
const promisifyAll = decorateAll(promisify);
```

# Filtering Objects

You might have already guessed, but if you can `map` an `Object`, you can also `filter` and Object.

```javascript
import filter from "ramda/src/filter";
import is from "ramda/src/is";
import anyPass from "ramda/src/anyPass";

const cat = {
  name: "mojo",
  age: 9,
  meow: () => console.log("meow")
};

const isPrimitive = anyPass([is(Boolean), is(Number), is(String)]);
const toModel = filter(isPrimitive);

toModel(cat);
// => { name: "mojo", age: 9 }
```

I won't spend too much time on filtering, the same techniques from `map` can be applied to `filter`, so I feel like it would just be redundant. Let's move on.

# Breaking early

There are times when you only need to iterate over the part of the collection. In a `for` loop you can break the iteration with the `break` command. Well, `ramda` has `transduce` which will let you do something similar.

note: transducers are an entire topic of their own, I'll barely be scratching the surface of transducers. For more info about transducers, I'd recommend starting here: [Understanding Transducers in JavaScript](https://medium.com/@roman01la/understanding-transducers-in-javascript-3500d3bd9624).

So let's see some code...

```javascript
const isOdd = x => x % 2 === 1;

// take the first 3 odd numbers
const firstThreeOddNumbers = compose(
  filter(isOdd),
  take(3)
);

// array values 0 to 100
const oneToOneHundred = range(0, 100);

// apply firstThreeOddNumbers to oneToOneHundred
transduce(firstThreeOddNumbers, (acc, x) => acc.concat(x), [], oneToOneHundred);
//=> [1, 3, 5]
```

Instead of iterating over the entire `0-100` range, the `transducer` iterates over `oneToOneHundred`, filtering them by `isOdd` and stopping after finding the first 3.

Check out Ramda's documentation to learn more about [transduce](https://ramdajs.com/docs/#transduce).

# Reduce

Normalizing data is a common task for large React applications using Redux. `reduce` is a perfect tool for data normalization.

Here's some typical data returned from an API.

```json
[
  { "id": 1, "value": "a" },
  { "id": 2, "value": "b" },
  { "id": 3, "value": "c" }
]
```

But I want to normalize it into something more React friendly. Something like this:

```json
{
  "byId": {
    "1": { "id": 1, "value": "a" },
    "2": { "id": 2, "value": "b" },
    "3": { "id": 3, "value": "c" }
  },
  "ids": [1, 2, 3]
}
```

Here's a reducer that will normalize the API data into the format I need.

```javascript
// before Ramda
const normalize = key => items =>
  items.reduce(
    (acc, item) => {
      acc.byId[item[key]] = item;
      acc.ids.push(item[key]);
      return acc;
    },
    { byId: {}, ids: [] }
  );
```

We can write this in a point-free style with Ramda's `reduce`.

```javascript
// point free style with Ramda's reduce
const normalize = key =>
  reduce(
    (acc, item) => {
      acc.byId[item[key]] = item;
      acc.ids.push(item[key]);
      return acc;
    },
    { byId: {}, ids: [] }
  );
```

There's not too much different. We just got rid of a couple references to `items`. Let's put a pin in this for now. We will use this as a base and build upon this example more later.

Just a note: The reducer could also be written like the code below, but I would like to demonstrate something that returns `acc`.

```javascript
(acc, item) => ({
  ...acc,
  byId: { ...acc.byId, [item[key]]: item },
  ids: acc.ids.concat(item[key])
});
```

# Tap

`tap` is one of my favorite functions. I tend to use it when I need to debug, mutate, or execute side-effects.

`tap` is a simple function, it executes a function and then returns the original argument.

It's probably easier to understand if I show you with code.

```javascript
// for demonstration purposes only
const tap = func => x => {
  func(x);
  return x;
};
```

Let's start with a basic `map`.

```javascript
const double = x => x * 2;

const inputs = [1, 2, 3];
inputs.map(double);
// => [2, 4, 6]
```

and add some `tap` style logging...

```javascript
import tap from "ramda/src/tap";

// without tap, console.log would have returned undefined.
// with tap, x will be returned.
const log = tap(x => console.log(x));
const double = x => x * 2;

const inputs = [1, 2, 3];
inputs.map(log).map(double);
// => 1
// => 2
// => 3
// => [2, 4, 6]
```

You can even use `tap` with a `Promise`.

```javascript
fetch("https://swapi.co/api/people/1")
  .then(response => response.json())
  .then(log)
  .then(processResponse);
```

# Tap2

The traditional `tap` function accepts only a single argument. This isn't very useful for 2 argument functions like `reduce`. This is easy to solve, we'll just create an enhanced `tap` function that can accept multiple arguments.

```javascript
// multi-argument tap
const tap2 = func => (...args) => (func(...args), args[0]);
```

Now we can eliminate the `return` statement from our reducer.

```javascript
const normalize = id =>
  reduce(
    tap2((acc, item) => {
      acc.byId[item[id]] = item;
      acc.ids.push(item[id]);
    }),
    { byId: {}, ids: [] }
  );
```

# Breaking early from reduce

Ramda has a special function called `reduced`. If you return a value wrapped in `reduced`, the reducer will break early.

```javascript
const values = [1, 2, 3, 4, 5];

// add all values. stop when total > 5
reduce((acc, item) => (acc > 5 ? reduced(acc) : acc + item), 0, values);
// => 6
```
