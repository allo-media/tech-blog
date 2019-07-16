---
layout: post
title:  "Stateful components in Elm"
excerpt: "It's often claimed that Elm developers should avoid thinking their views as stateful components. While this is indeed a general best design practice, sometimes you may want to make your views reusable, and if they come with a state... you end up copying and pasting a lot of things."
date:   2019-07-16 10:00:00 +0100
categories: elm
tags: elm
---

It's often claimed that [Elm] developers should avoid thinking their views as stateful components. While this is indeed a general best design practice, sometimes you may want to make your views reusable (eg. across pages or projects), and if they come with a state... you end up copying and pasting a lot of things.  

We recently published [elm-daterange-picker], a date range picker written in [Elm]. It was the perfect occasion investigating what a reasonable API for a reusable stateful view component would look like.

![widget demo](blob:https://imgur.com/d30c10d3-4f74-4b24-8d5a-423fa3b5eac2)

Many component/widget-oriented Elm packages feature a rather raw [TEA] API, namely exposing `Model`, `Msg(..)`, `init`, `update` and `view`, so you can basically import what defines an actual application and embed it within your own application.   

![funny meme](https://i.imgur.com/aBXjI9N.png)

With these, you usually end up writing things like this:

```elm
import Counter


type alias Model =
    { widget : Counter.Model
    , value : Maybe Int
    }


type Msg
    = CounterMsg Counter.Msg


init : () -> ( Model, Cmd Msg )
init _ =
    ( { widget = Counter.init, value = Nothing }
    , Cmd.none
    )


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        CounterMsg widgetMsg ->
            let
                ( newCounterModel, newCounterCommands ) =
                    Counter.update widgetMsg
            in
            ( { model
                | widget = newCounterModel
                , value =
                    case widgetMsg of
                        Counter.Apply value ->
                            Just value

                        _ ->
                            Nothing
              }
            , newCommands |> Cmd.map CounterMsg
            )


view : Model -> Html Msg
view model =
    div []
        [ Counter.view model.widget
            |> Html.map CounterMsg
        , text (String.fromInt model.value)
        ]
```

This certainly works, but let's be frank for a minute and admit this is super verbose and not very developer friendly:

- You need to `Cmd.map` and `Html.map` here and there
- You need to pattern match `Widget.Msg` to intercept whatever event interests you...
- ... meaning `Widget` exposes all `Msg`s, which are **implementation details** you now rely on.

There's another way, which [Evan] explained in his now deprecated [elm-sortable-table] package. Among the many good points he has, one idea stroke me as brilliantly simple yet effective to simplify such stateful view components API design: **state updates can be managed right from event handlers**.

Let's imagine a simple counter; what if when clicking the *increment* button, instead of calling `onClick` with some `Increment` message, we would call a generic one with the new counter state updated accordingly?

```elm
-- Widget.elm
view : (Int -> msg) -> Int -> Html msg
view toMsg counter =
    button [ onClick (toMsg (counter + 1)) ]
        [ text "increment" ]
```

Or if you want to use an [opaque type], which is an excellent idea for maintaining the smallest API surface area:

```elm
-- Widget.elm
type State
    = State Int

view : (State -> msg) -> State -> Html msg
view toMsg (State value) =
    button [ onClick (toMsg (State (value + 1))) ]
        [ text "increment" ]
```

So for instance, to use this `Counter` component in your own application, you just have to write this:

```elm
type alias Model =
    { widget : Counter.State
    , value : Maybe Int
    }


type Msg
    = CounterChanged Counter.State


init : () -> ( Model, Cmd Msg )
init _ =
    ( { widget = Counter.init, value = Nothing }
    , Cmd.none
    )


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        CounterChanged state ->
            ( { model | counter = state, value = Counter.value state }
            , Cmd.none
            )


view : Model -> Html Msg
view model =
    div []
        [ Counter.view CounterChanged model.widget
        , text (String.fromInt model.value)
        ]
```

Notice how our `update` function is dramatically simpler to write, and to understand.

Of course a counter wouldn't be worth creating a package for it, though this may highlight the concept better. Don't hesitate reading *elm-daterange-picker*'s [source code] and [demo code] to look at a real world application of this design principle.

[Elm]: https://elm-lang.org/
[Evan]: https://github.com/evancz/
[demo code]: https://github.com/allo-media/elm-daterange-picker/blob/master/demo/Main.elm
[elm-daterange-picker]: https://package.elm-lang.org/packages/allo-media/elm-daterange-picker/latest/
[elm-sortable-table]: https://github.com/evancz/elm-sortable-table#about-api-design
[opaque type]: https://medium.com/@ckoster22/advanced-types-in-elm-opaque-types-ec5ec3b84ed2
[source code]: https://github.com/allo-media/elm-daterange-picker/blob/master/src/DateRangePicker.elm
[TEA]: https://guide.elm-lang.org/architecture/
