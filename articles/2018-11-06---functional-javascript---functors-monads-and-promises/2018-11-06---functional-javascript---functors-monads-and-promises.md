# Functional JavaScript - Functors, Monads, and Promises

[![Person holding a box wrapped in ribbon](https://thepracticaldev.s3.amazonaws.com/i/l3jaky9jm4lvifezynjt.jpg)](https://www.pexels.com/photo/person-s-holds-brown-gift-box-842876)

Some people have said a `Promise` is a `Monad`. Others have said a `Promise` is not a `Monad`. They are both wrong... and they are both right.

By the time you finish reading this article, you will understand what a `Functor` and `Monad` are and how they are similar and different from a `Promise`.

# Why can't anyone explain a Monad?

It is difficult to explain what a Monad is without also having the prerequisite vocabulary also required to understand it.

I love this video with Richard Feynman when he is asked to describe "what is going on" between two magnets.

The whole video is amazing and mind blowing, but you can skip straight to 6:09 if you have some sort of aversion to learning.

{% youtube MO0r930Sn_8 %}

> I can't explain that attraction in terms of anything else that's familiar to you - Richard Feynman @ [6:09](https://www.youtube.com/watch?v=MO0r930Sn_8#t=6m9s)

So let's backup a few steps and learn the vocabulary required to understand what a `Monad` is.

# Are we ready to understand a Functor?

Definition: A `Functor` is something that is `Mappable` or something that can be mapped between Categories.

Okay... Not yet. But do not be afraid, you are already familiar with `Functors` if you have used `Array`'s `map` function.

```javascript
[1, 2, 3].map(x => x * 2) //=> [2, 4, 6]
```

Before we can fully understand a `Functor`, we also have to understand what it means to be `Mappable` and to understand that we also have to understand what a `Category` is. So let's begin there.

# Categories and Maps

![Category theory triangle](https://thepracticaldev.s3.amazonaws.com/i/odihwyfbo86x93917533.png)

A `category` consists of a collection of things. This could be numbers, strings, urls, customers, or any other way you wish to organize like-things. (X, Y, and Z in the graphic are the Categories.)

A `map` is a function to convert something from one category to another (also itself). (f, g, and fog are the maps). 🔍 Google tip: A `map` between Categories is called a `Morphism`.

Example: An object in the category `Number` can be converted into the category `String` using the `toString()` method.

```javascript
// A map of Number -> String
const numberToString = num => num.toString()
```

You can also create `maps` back into their own category or more complex categories.

```javascript
// A map of Number -> Number
const double => num => num * 2

// A map of Array -> Number
const arrayToLength = array => array.length

// A map of URL -> Promise (JSON)
const urlToJson = url =>
  fetch(url)
    .then(response => response.json())
```

So a category could be simple like a Number or a String. A category could also be more abstract like a Username, A User API URL, User API HTTP Request, User API Response, User API Response JSON. Then we can create maps or morphisms between each category to get the data we want.

Examples of morphisms:
- Username -> User API Url
- User API Url -> User API HTTP Request
- User API HTTP Request -> User API Response
- User API Response -> User API Response JSON

🔍 Google tip: `Function Composition` is a way to combining multiple `map` or `morphisms` to create new `maps`. Using `Function Composition` we could create a map from `Username` directly to `User API Response JSON`

# Back to the Functor

Now that we understand what it means to be `Mappable`, we can finally understand what a `Functor` is.

A `Functor` is something that is `Mappable` or something that can be mapped between Categories.

An `Array` is `Mappable`, so it is a `Functor`. In this example I am taking an `Array of Numbers` and morphing it into an `Array of Strings`.

```javascript
const numberToString = num => num.toString()

const array = [1, 2, 3]
array.map(numberToString)
//=> ["1", "2", "3"]
```

Note: One of the properties of a `Functor` is that they always stay that same type of `Functor`. You can morph an `Array` containing `Strings` to `Numbers` or any other object, but the `map` will ensure that it will always be an `Array`. You cannot `map` an `Array` of `Number` to just a `Number`.

We can extend this `Mappable` usefulness to other objects too! Let's take this simple example of a `Thing`.

```javascript
const Thing = value => ({
  value
})
```

If we wanted to make `Thing` mappable in the same way that `Array` is mappable, all we have to do is give it a `map` function.


```javascript
const Thing = value => ({
  value,
  map: morphism => Thing(morphism(value))
//                 ----- -------- -----
//                /        |            \
// always a Thing          |             value to be morphed
//                         |
//             Morphism passed into map
})

const thing1 = Thing(1)               // { value: 1 }
const thing2 = thing1.map(x => x + 1) // { value: 2 }
```

And that is a `Functor`! It really is just that simple.

![Thing 1 and Thing 2 from Dr Seuse](https://thepracticaldev.s3.amazonaws.com/i/s5zz0ecd5opxfcxp7nj0.jpg)

🔍 Google tip: The `"Thing"` `Functor` we created is known as `Identity`.

# Back to the Monad

Sometimes functions return a value already wrapped. This could be inconvenient to use with a `Functor` because it will re-wrap the `Functor` in another `Functor`.

```javascript
const getThing = () => Thing(2)

const thing1 = Thing(1)

thing1.map(getThing) //=> Thing (Thing ("Thing 2"))
```

This behavior is identical to `Array`'s behavior.

```javascript
const doSomething = x => [x, x + 100]
const list = [1, 2, 3]

list.map(doSomething) //=> [[1, 101], [2, 102], [3, 103]]
```

This is where `flatMap` comes in handy. It's similar to `map`, except the morphism is also expected to perform the work of wrapping the value.

```javascript
const Thing = value => ({
  value,
  map: morphism => Thing(morphism(value)),
  flatMap: morphism => morphism(value)
})

const thing1 = Thing(1)                          //=> Thing (1)
const thing2 = thing1.flatMap(x => Thing(x + 1)) //=> Thing (2)
```

That looks better!

This could come in handy in a `Maybe` when you might need to switch from a `Just` to a `Nothing`, when for example a prop is missing.

```javascript
import Just from 'mojiscript/type/Just'
import Nothing from 'mojiscript/type/Nothing'

const prop = (prop, obj) =>
  prop in obj
    ? Just(obj[prop])
    : Nothing

Just({ name: 'Moji' }).flatMap(x => prop('name', x)) //=> Just ("Moji")
Just({}).flatMap(x => prop('name', x))               //=> Nothing
```

This code could be shortened to:

```javascript
const Just = require('mojiscript/type/Just')
const Nothing = require('mojiscript/type/Nothing')
const { fromNullable } = require('mojiscript/type/Maybe')

const prop = prop => obj => fromNullable(obj[prop])

Just({ name: 'Moji' }).flatMap(prop('name')) //=> Just ("Moji")
Just({}).flatMap(prop('name'))               //=> Nothing
```

🔍 Google tip: This code shortening is made possible with `currying`, `partial application`, and a `point-free style`.

I hope at this point you are thinking this was an easier journey than you initially thought it would be. We have covered `Functors` and `Monads` and next up in the `Promise`!

# The Promise

If any of that code looks familiar it's because the `Promise` behaves like both `map` and `flatMap`.

```javascript
const double = num => num * 2

const thing1 = Thing(1)             //=> Thing (1)
const promise1 = Promise.resolve(1) //=> Promise (1)

thing1.map(double)    //=> Thing (2)
promise1.then(double) //=> Promise (2)

thing1.flatMap(x => Thing(double(x)))          //=> Thing (2)
promise1.then(x => Promise.resolve(double(x))) //=> Promise (2)
```

As you can see the `Promise` method `then` works like `map` when an unwrapped value is returned and works like `flatMap`, when it is wrapped in a `Promise`. In this way a `Promise` is similar to both a `Functor` and a `Monad`.

This is also the same way it differs.

```javascript
thing1.map(x => Thing(x + 1))              // Thing (Thing (2))
promise1.then(x => Promise.resolve(x + 1)) // Promise (2)

thing1.flatMap(x => x + 1) //=> 2
promise1.then(x => x + 1)  //=> Promise (2)
```

If I wanted to wrap a value twice (think nested `Arrays`) or control the return type, I am unable to with `Promise`. In this way, it breaks the `Functor` laws and also breaks the `Monad` laws.

# Summary

- A `Functor` is something that is `Mappable` or something that can be mapped between Categories.
- A `Monad` is similar to a `Functor`, but is `Flat Mappable` between Categories.
- `flatMap` is similar to `map`, but yields control of the wrapping of the return type to the mapping function.
- A Promise breaks the `Functor` and `Monad` laws, but still has a lot of similarities. Same same but different.

Continue reading: [NULL, "The Billion Dollar Mistake", Maybe Just Nothing](https://dev.to/joelnet/null-the-billion-dollar-mistake-maybe-just-nothing-1cak)

My articles show massive Functional JavaScript love. If you need more FP, follow me here or on Twitter [@joelnet](https://twitter.com/joelnet)!

And thanks to my buddy Joon for proofing this :)

![Cheers!](https://thepracticaldev.s3.amazonaws.com/i/6jsy3a866frzp3u5oda0.jpg)
