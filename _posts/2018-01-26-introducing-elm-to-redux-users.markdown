---
layout: post
title:  "Introducing Elm to Redux users"
date:   2018-01-26 17:53:00 +0100
categories: elm react javascript
---

I recently had to introduce some [Elm] concepts to a coworker who had some experience with [React] and [Redux]. One of these concepts was [List.foldl], a reduction function which exists in many languages, specifically as [Array#reduce] in JavaScript.

The coworker was struggling to understand the whole concept, so I tried to use a metaphor; I came with the idea of a [Ferris wheel] next to a lake, with someone in one of its basket holding a bucket, filling the basket with water from the lake everytime the basket was back to the ground.

<a href="https://www.flickr.com/photos/45909111@N00/3812448452/in/photolist-6NTPpE-6NTN6S-8VeQqf-6NTMNw-fmHhYZ-fmHg7k-atTbFs-8VeK2S-atQvRT-6NTKw1-aUDvNP-7dfSYz-2XhKZV-fmXsCG-fmXscE-4sTcSG-8VeBij-fmHhsP-wMJMfg-wuBR7h-wuKc1H-wuBTjf-vQdnbN-wMeCvV-wMJKda-NoBSUY-NvJoFw-MAWVsx-NoBSPC-NoBSxL-NvJoKE-NoBSJY-NvJoFb-NoBSFw-NvJoBJ-MAWP4F-NvJqem-MAWvqe-NvJoHW-MAWv1M-NvJoMd-MAWvcP-vQmVTV-NyVKgD-wuKeKk-wuKdhF-wuBQ57-8VePww-8VbGbk-8Vbz14/" title="Gwydion M. Williams - View of Wuxi from Lake Tai">
    <img src="https://farm3.staticflickr.com/2509/3812448452_c6ecd0424f_z.jpg"
    width="640" height="463" alt="Gwydion M. Williams - View of Wuxi from Lake Tai">
</a>

Yeah, I know.

So as he was starring at me like I was a crazy person, and as I knew he did use React and Redux in the past, I told him it was like the [*reducer functions*](https://redux.js.org/docs/basics/Reducers.html) he probably used already.

We started with a silly reimplementation of redux' reducers in plain js:

```js
const init = {water: 0}

const actions = [
    {type: "ADD_WATER"},
    {type: "ADD_WATER"},
    {type: "ADD_WATER"},
]

function reducer(state, action) {
    switch(action.type) {
        "EMPTY": {
            return init
        },
        "ADD_WATER": {
            return {...state, water: state.water + 1}
        }
    }
}

const state = actions.reduce(reducer, init)

console.log(state) // {water: 3}
```

He was like "oh yeah, I know that". Good!

So I could use the Ferris wheel metaphor again:

- `state` represents the state of the wheel basket (and the quatity of water in it)
- `init` is the initial state of the wheel basket (it contains no water yet)
- `actions` are the list of wheel full rotations where the basket reach the ground again, with the operation to proceed (here, filling the basket with water from the lake)

For the records, yes my coworker was still very oddly looking at me.

We moved on and decided to reimplement the same thing in Elm, using `foldl`. Its type signature is:

```elm
foldl : (a -> b -> b) -> b -> List a -> b
```

Wow, that looks complicated, especially when you're new to Elm. Let's decompose:

- `(a -> b -> b)` means we want a function, taking two arguments `a` and `b` and returning `b`. That sounds a lot like our `reducer` function in JavaScript! If so, `a` is an action, and `b` a state.
- the next argument, `b`, is the same type as what we must return from the function passed as the first argument; so that means a state! It's probably the initial state we start reducing our list of actions from.
- the next argument, `List a`, is certainly our list of actions.
- And all this must return a `b`, hence a new state. We have the exact definition of what we're after.

```elm
type Action
    = AddWater
    | Empty

type alias State =
    { water : Int }

init : State
init =
    { water = 0 }

actions : List Action
actions =
    [ AddWater, AddWater, AddWater ]

reducer : Action -> State -> State
reducer action state =
    case action of
        Empty ->
            init

        AddWater ->
            { state | water = state.water + 1 }

update : List Action -> State -> State
update actions state =
    List.foldr reducer state actions

main =
    text <| toString (update actions init) -- {water: 3}
```

We quickly drafted this [on Ellie](https://ellie-app.com/kL3dJS7Gta1/1). It's not graphically impressive, but it works.

That was it, it was more obvious how to map things my coworker already knew to something new to him, while in fact it was actually exactly the same thing, expressed slightly differently from a syntax perspective.

We also expanded that the [Elm Architecture] was basically just a projection of our own `update` function (and so, of the `foldl` one), `Action` being usually named *Msg* and `State` *Model*.

The funny thing being, Redux design itself was initially inspired by Elm!

In conclusion, here are three tips when facing something difficult to understand:

- start with finding a metaphor, even a silly one; that helps summarizing the problem and ensure you get the big picture of it;
- slice the problem down to the smallest understandable chunks you can, then move to the next larger one when you're done;
- always try to map what you're trying to learn to things you've already learned; past experiences are good tools for that.

[Array#reduce]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
[Elm]: http://elm-lang.org/
[Elm Architecture]: https://guide.elm-lang.org/architecture/
[Ferris wheel]: https://en.wikipedia.org/wiki/Ferris_wheel
[List.foldl]: http://package.elm-lang.org/packages/elm-lang/core/latest/List#foldl
[React]: https://reactjs.org/
[Redux]: https://redux.js.org/
