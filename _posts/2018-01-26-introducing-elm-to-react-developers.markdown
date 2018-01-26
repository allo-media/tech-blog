---
layout: post
title:  "Introducing Elm to React developers"
date:   2018-01-26 17:53:00 +0100
categories: elm react javascript
---

I recently had to introduce some Elm concepts to a coworker who has some experience with React. One of these concepts was `foldl`, the reduction function which exists in many languages, specifically as `Array#reduce` in JavaScript.

The coworker was puzzling understanding the concept, so I told him it was a bit like the *reducers* functions from Redux.

We started with a silly reimplementation of redux' reducers in plain js:

```js
function reducer(action, state) {
    switch(action.type) {
        "ADD_WATER": {
            return {...state, water: state.water + 1};
        }
    }
}

actions = [
    {type: "ADD_WATER"},
    {type: "ADD_WATER"},
    {type: "ADD_WATER"}
]

const state = actions.reduce((state, action) => reducer(action, state), {water: 0})
console.log(state) // {water: 3}
```

He was like "oh yeah, I know that". Good! Then I suggested we reimplement the very same thing in Elm:

```elm
type Action
    = AddWater
    | NoOp


type alias State =
    { water : Int }


actions =
    [ AddWater, AddWater, AddWater ]


reducer : Action -> State -> State
reducer action state =
    case action of
        NoOp ->
            state

        AddWater ->
            { state | water = state.water + 1 }


update : List Action -> State -> State
update actions state =
    List.foldr reducer state actions

main =
    text <| toString (update actions { water = 0 }) -- {water: 3}
```

The Ellie sample is available at https://ellie-app.com/kL3dJS7Gta1/0

That was it, it was more obvious how to map things he already knew to something new, which was actually exactly the same thing. The funny thing being, Redux was initially inspired by Elm!  
