# NULL, "The Billion Dollar Mistake", Maybe Just Nothing

![Joker burning a huge pile of money](https://thepracticaldev.s3.amazonaws.com/i/c8q2xf9cx0wh2le4n3v0.jpg)

Tony Hoare, the creator of NULL, now refers to NULL as The Billion Dollar Mistake. Even though NULL Reference Exceptions continue to haunt our code to this day, we still choose to continue using it.

And for some reason JavaScript decided to double down on the problems with `null` by also creating `undefined`.

Today I would like to demonstrate a solution to this problem with the Maybe.

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years. -- [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare)

# Do not underestimate the problems of NULL

Before you have even finish reading this article... I can already sense it, your desire to hit PAGE DOWN, rush straight to the comment section and blast out a "but NULL is never a problem for ME". But please pause, slow down, read and contemplate.

8 of 10 errors from [Top 10 JavaScript errors from 1000+ projects (and how to avoid them)](https://rollbar.com/blog/top-10-javascript-errors/) are `null` and `undefined` problems. Eight. Out. Of. Ten.

To underestimate NULL is to be defeated by NULL.

# Null Guards

Because of the problems `null` brings with it, we have to constantly guard our code from it. Unguarded code might look something like this:

```javascript
const toUpper = string => string.toUpperCase()
```

This code is susceptible to NULL Reference Exceptions.

```javascript
toUpper(null) //=> ​​Cannot read property 'toUpperCase' of null​​
```

So we are forced to guard against `null`.

```javascript
const toUpper = string => {
  if (string != null) {
//    --------------
//                   \
//                    null guard
    return string.toUpperCase()
  }
}
```

But this quickly becomes verbose as everywhere that may encounter `null` has to be guarded.

```javascript
const toUpper = string => {
  if (string != null) {
//    --------------
//                   \
//                    duplication
    return string.toUpperCase()
  }
}

const toLower = string => {
  if (string != null) {
//    --------------
//                   \
//                    duplication
    return string.toLowerCase()
  }
}

const trim = string => {
  if (string != null) {
//    --------------
//                   \
//                    duplication
    return string.trim()
  }
}
```

If we think about a values as having a one-to-many relationship with code that may access it, then it makes more sense to place the guards on the _one_ and not on the _many_.

# Nullable Types

The .NET Framework 2.0 introduced Nullable Types into the .NET language. This new Nullable value, could be set to null without the reference being null. This meant if `x` was a Nullable Type, you could still do things like `x.HasValue` and `x.Value` without getting a `NullReferenceException`.

```csharp
int? x = null
if (x.HasValue)
{
    Console.WriteLine($"x is {x.Value}")
}
else
{
    Console.WriteLine("x does not have a value")
}
```

# The Maybe

The `Maybe` is similar to a Nullable Type. The variable will always have a value, and that value might represent a `null`, but it will never be set to `null`.

For these examples, I'll be using the `Maybe` from MojiScript. (Also checkout [monet](https://github.com/monet/monet.js/blob/develop/docs/MAYBE.md) and [Sanctuary](https://sanctuary.js.org/#maybe-type), [Folktale](https://github.com/folktale/data.maybe) for other `Maybes`). Use the following import:

```javascript
import { fromNullable } from "mojiscript/type/Maybe"
```

The `Maybe` is a union type of either a `Just` or a `Nothing`. `Just` contains a value and `Nothing` is well... nothing.

But now the value is all wrapped up inside of the `Maybe`. To access the value of a `Maybe`, you would have touse a `map` function. [Fun to Google](https://www.google.com/search?q=javascript+functor): `map` is what makes the `Maybe` type a `Functor`.

If you are getting that feeling that you have seen this somewhere before that is because this exactly how a `Promise` works. The difference is `Promise` uses `then` and `Maybe` uses `Map`.

```javascript
const promise = Promise.resolve(888)
const maybe = Just(888)

promise.then(double)
maybe.map(double)
```

Same same but different.

```javascript
const toUpper = string => string.toUpperCase()

Just("abc").map(toUpper) //=> Just ('ABC')
Nothing.map(toUpper) //=> Nothing
```

Notice how in both cases above, the `toUpper` function no longer throws an `Error`. That is because we are no longer calling `toUpper` directly with a `String`, but instead mapping it with our `Maybe`.

If we convert all types within our application to use a `Maybe`, then all null guards are no longer necessary.

The `null` is now guarded in a single place, in the `Maybe` type, instead of being sprinkled throughout the application, wherever the value might be accessed.

The `Maybe` is a guard on the _one_ instead of the _many_!

![Neo vs many agent Smiths](https://thepracticaldev.s3.amazonaws.com/i/8i3rzn0tm7y9qepja9s4.jpg)

# Getting in and out of Maybes

But what about the times when we are not in control of the code, when we must send or receive a `null` value? Some examples might be 3rd party libraries that will return a `null` or libraries that will require passing `null` as an argument.

In these cases, we can convert a null value to a Maybe using `fromNullable` and we can convert back to a nullable value using `fromMaybe`.

```javascript
import { fromMaybe, fromNullable } from "mojiscript/type/Maybe"

// converting nullable values to a Maybe
fromNullable(undefined) //=> Nothing
fromNullable(null) //=> Nothing
fromNullable(123) //=> Just (123)
fromNullable("abc") //=> Just ("abc")

// converting Maybe to a nullable type
fromMaybe(Just("abc")) //=> 'abc'
fromMaybe(Nothing) //=> null
```

You could als guard a single function like this:

```javascript
const toUpper = string =>
  fromNullable(string).map(s => s.toUpperCase()).value
```

But that is a little verbose and it's much better to expand the safety of the Maybe type to the entire application. Put the guards in place at the gateways in and out of your application, not individual functions.

One example could be using a Maybe in your Redux.

```javascript
// username is a Maybe, initially set to Nothing.
const initalState = {
  username: Nothing
}

// your reducer is the gateway that ensures the value will always be a maybe.
const reducer = (state = initialState, { type, value }) =>
  type === 'SET_USERNAME'
    ? { ...state, username: fromNullable(value) }
    : state

// somewhere in your render
render() {
  const userBlock = this.props.username.map(username => <h1>{username}</h1>)
  const noUserBlock = <div>Anonymous</div>

  return (
    <div>
    {fromMaybe (noUserBlock) (userBlock)}
    </div>
  )
}
```

# JavaScript Type Coercion

MojiScript's `Maybe` can use JavaScript's implicit and explicit coercion to it's advantage.

`Maybe` can be implicity coerced into a `String`.

```javascript
// coercing to a String
console.log("a" + Just("b") + "c") //=> 'abc'
console.log("a" + Nothing + "c") //=> 'ac'
```

`Maybe` can be explicity coerced into a `Number`.

```javascript
Number(Just(888)) //=> 888
Number(Nothing) //=> 0
```

`Maybe` can even be stringified.

```javascript
const data = {
  id: Nothing,
  name: Just("Joel")
}

JSON.stringify(data)
//=> {"id":null,"name":"Joel"}
```

# Accessing Nested Objects

Let's take a look at the common task of accessing nested objects.

We'll use these objects. One is lacking an address, which can yield `nulls`. Gross.

```javascript
const user1 = {
  id: 100,
  address: {
    address1: "123 Fake st",
    state: "CA"
  }
}

const user2 = {
  id: 101
}
```

These are common ways to access nested objects.

```javascript
user1.address.state //=> 'CA'
user2.address.state //=> Error: Cannot read property 'state' of undefined

// short circuit
user2 && user2.address && user2.address.state //=> undefined

// Oliver Steel's Nested Object Pattern
((user2||{}).address||{}).state //=> undefined
```

Prettier seems to hate both of those techniques, turning them into unreadable junk.

Now let's try accessing nested objects with a `Maybe`.

```javascript
import { fromNullable } from 'mojiscript/type/Maybe'

const prop = prop => obj =>
  fromNullable(obj).flatMap(o => fromNullable(o[prop]))

Just(user1)
  .flatMap(prop('address))
  .flatMap(prop('state)) //=> Just ("CA")

Just(user2)
  .flatMap(prop('address))
  .flatMap(prop('address)) //=> Nothing
```

A lot of this boiler plate can be reduced with some helper methods.

```javascript
import pathOr from 'mojiscript/object/PathOr'
import { fromNullable } from 'mojiscript/type/Maybe'

const getStateFromUser = obj =>
  fromNullable(pathOr (null) ([ 'address', 'state' ]) (obj))

Just(user1).map(getStateFromUser) //=> Just ("CA")
Just(user2).map(getStateFromUser) //=> Nothing
```

# Decoupled map function

A Map can also be decoupled from `Maybe`. There are many libs that have a `map` function, like Ramda, but I'll be using the one from [MojiScript](https://github.com/joelnet/MojiScript) for this example.

```javascript
import map from 'mojiscript/list/map'

const toUpper = string => string.toUpperCase()

Just("abc").map(toUpper) //=> Just ('ABC')
Nothing.map(toUpper) //=> Nothing
```

```javascript
import map from 'mojiscript/list/map'

const toUpper = string => string.toUpperCase()

map (toUpper) (Just ("abc")) //=> Just ('ABC')
map (toUpper) (Nothing) //=> Nothing
```

This was getting far too big for this section, so it has been broken out into it's own article here: [An introduction to MojiScript's enhanced map](https://dev.to/joelnet/an-introduction-to-mojiscripts-enhanced-map-3dpp)

# Heavy Lifting

Lifting is a technique to apply `Applicatives` to a function. In English that means we can use "normal" functions with our `Maybes`. Fun to Google: `ap` is what makes the `Maybe` type an [`Applicative`](https://www.google.com/search?q=javascript+applicative).

This code will use `liftA2`, `A` for `Applicative` and `2` for the number of arguments in the function.

```javascript
import liftA2 from "mojiscript/function/liftA2"
import Just from "mojiscript/type/Just"
import Nothing from "mojiscript/type/Nothing"

const add = x => y => x + y
const ladd = liftA2 (add)

add (123) (765) //=> 888

ladd (Just (123)) (Just (765)) //=> Just (888)
ladd (Nothing) (Just (765)) //=> Nothing
ladd (Just (123)) (Nothing) //=> Nothing
```

Some things to notice:

- The function `add` is curried. You can use any `curry` function to do this for you.
- `add` consists of 2 parameters. If it was 3, we would use `liftA3`.
- All arguments must be a `Just`, otherwise `Nothing` is returned.

So now we do not have to modify our functions to understand the `Maybe` type, we can use `map` and also `lift` to apply the function to our `Maybes`.

Continue Learning: [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) does an incredible job of explaining this and more!

# Maybe Function Decorator

There are times when you would like to guard a single function against NULL. That is where the `maybe` Function Decorator comes in handy.

```javascript
const maybe = func => (...args) =>
  !args.length || args.some(x => x == null)
    ? null
    : func(...args)
```

Guard your functions against null with the `maybe` function decorator:

```javascript
const toUpper = string => string.toUpperCase()
const maybeToUpper = maybe(toUpper)
maybeToUpper("abc") //=> 'ABC'
maybeToUpper(null) //=> null
```

Can also be written like this:

```javascript
const toUpper = maybe(string => string.toUpperCase())
```

Learn more about Function Decorators:

- [Function decorators: Transforming callbacks into promises and back again](e274c7cf7293)
- [Functional JavaScript: Function Decorators Part 2](https://dev.to/joelnet/function-decorators-part-2-javascript-4km9)

# TC39 Optional Chaining for JavaScript

This is a good time to mention the [TC39 Optional Chaining Proposal](https://github.com/tc39/proposal-optional-chaining) that is currently in Stage 1.

Optional Chaining will allow you to guard against null with a shorter syntax.

```javascript
// without Optional Chaining
const toUpper = string => string && string.toUpperCase()

// with Optional Chaining
const toUpper = string => string?.toUpperCase()
```

Even with Optional Chaining, the guards are still on the _many_ and not the _one_, but at least the syntax is short.

# Wisdoms

- To underestimate NULL is to be defeated by NULL.
- 8 out of the 10 top 10 errors are NULL and undefined errors.
- If we think about a values as having a one-to-many relationship with code that may access it, then it makes more sense to place the guards on the _one_ and not on the _many_.
- It is possible to completely eliminate an entire class of bugs (NULL Reference Exceptions) by eliminating `null`.
- Having NULL Reference Exceptions in your code is a choice.

# End

Have questions or comments? I'd love to hear them!

[Hop over to the MojiScript Discord chat and say hi!](https://discord.gg/Gg7ptD5)

This turned out a little longer than I originally thought it would. But this is a subject that is hard to sum up into a single article.

You can also use the `Maybe` with MojiScript's `map`. Read more about how awesome MojiScript's map is here...

My articles are very Functional JavaScript heavy, if you need more FP, follow me here, or on Twitter [@joelnet](https://twitter.com/joelnet)!

![Cheers!](https://thepracticaldev.s3.amazonaws.com/i/6jsy3a866frzp3u5oda0.jpg)
