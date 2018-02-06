---
layout: post
title:  "Chaining HTTP requests in Elm"
excerpt: "Sometimes in Elm you struggle with the most basic things. Especially when you come from a JavaScript background, where chaining HTTP requests are relatively easy thanks to Promises or async/await."
date:   2018-02-05 09:00:00 +0100
categories: learning elm
tags: elm http
---

Sometimes in Elm you struggle with the most basic things.

Especially when you come from a JavaScript background, where chaining HTTP requests are relatively easy thanks to Promises. Here's a real-world example leveraging the Github public API, where we fetch a list of Github events, pick the first one and query some user information from its unique identifier.

The first request uses the `https://api.github.com/events` endpoint, and the JSON retrieved looks like this:

```json
[
    {
        "id": "987654321",
        "type": "ForkEvent",
        "actor": {
            "id": 1234567,
            "login": "foobar",
        }
    },
]
```

I'm purposely omitting a lot of other properties from the records here, for brevity.

The second request we need to do is on the `https://api.github.com/users/{login}` endpoint, and its payload looks like this:

```json
{
    "id": 1234567,
    "login": "foobar",
    "name": "Foo Bar",
}
```

Again, I'm just displaying a few fields from the actual JSON payload here.

So we basically want:

- from a list of events, to pick the first one if any,
- then pick its `actor.login` property,
- query the user details endpoint using this value,
- extract the user real name for that account.

Using JavaScript, that would look like this:

```js
fetch("https://api.github.com/events")
    .then(responseA => {
        return responseA.json()
    })
    .then(events => {
        if (events.length == 0) {
            throw "No events."
        }
        const { actor : { login } } = events[0]
        return fetch(`https://api.github.com/users/${login}`)
    })
    .then(responseB => {
        return responseB.json()
    })
    .then(user => {
        if (!user.name) {
            console.log("unspecified")
        } else {
            console.log(user.name)
        }
    })
    .catch(err => {
        console.error(err)
    })
```

It would get a little fancier using `async/await`:

```js
try {
    const responseA = await fetch("https://api.github.com/events")
    const events = await responseA.json()
    if (events.length == 0) {
        throw "No events."
    }
    const { actor: { login } } = events[0]
    const responseB = await fetch(`https://api.github.com/users/${login}`)
    const user = await responseB.json()
    if (!user.name) {
        console.log("unspecified")
    } else {
        console.log(user.name)
    }
} catch (err) {
    console.error(err)
}
```

This is already complicated code to read and understand, and it's tricky to do using Elm as well. Let's see how to achieve the same understanding exactly what we're doing (we've all blindly copied and pasted code in the past, don't deny).

First, let's write the two requests we need; one for fetching the list of events, the second to obtain a given user's details from her `login`:

```haskell
import Http
import Json.Decode as Decode

eventsRequest : Http.Request (List String)
eventsRequest =
    Http.get "https://api.github.com/events"
        (Decode.at [ "actor", "login" ] Decode.string
            |> Decode.list
        )

nameRequest : String -> Http.Request String
nameRequest login =
    Http.get ("https://api.github.com/users/" ++ login)
        (Decode.at [ "name" ]
            (Decode.oneOf
                [ Decode.string
                , Decode.null "unspecified"
                ]
            )
        )
```

These two functions return `Http.Request` with the type of data they'll retrieve from the JSON body of their respective responses. `nameRequest` handles the case where a Github users don't have entered their real name yet, so the `name` filed might be a null; as with the JavaScript version, we then default to `"unspecified"`.

That's good but now we need to execute and chain these two requests, the second one depending on the result of the first one, where we retrieve the `actor.login` value of the event payload.

Elm is a pure language, meaning you can't have side effects in your functions (an HTTP request is a huge side effect). So your functions must return *something* that represents a given side effect, then another function must handle the result when a response is received.

In Elm, you're gonna use a [Task] to define and perform such asynchronous side effects, and tasks may `succeed` or `fail` (think `Promise.resolve` and `Promise.reject`). The [Http] package provides `Http.toTask` to map a request to a `Task`. Let's use that here:

```haskell
fetchEvents : Task Http.Error (List String)
fetchEvents =
    eventsRequest |> Http.toTask

fetchName : String -> Task Http.Error String
fetchName login =
    nameRequest login |> Http.toTask
```

I created these two simple functions mostly to focus on their return types; a `Task` must define an error type and a result type. For example, `fetchEvents` being an HTTP task, it will receive an `Http.Error` when the task fails, and a list of strings when the task succeeds.

But dealing granularily with HTTP errors being out of scope of this already long blog post, and in order to keep things as less convoluted as possible, I'm gonna use `Task.mapError` to turn complex HTTP error objects into strings as well:  


```haskell
toHttpTask : Http.Request a -> Task String a
toHttpTask request =
    request
        |> Http.toTask
        |> Task.mapError toString

fetchEvents : Task String (List String)
fetchEvents =
    toHttpTask eventsRequest

fetchName : String -> Task String String
fetchName login =
    toHttpTask (nameRequest login)
```

`toHttpTask` is a common helper turning an `Http.Request` into a `Task`, transforming the `Http.Error` complex type into a serialized, purely textual version of it.

We'll also need a function allowing to extract the very first element of a list, if any. Such a function is builtin the `List` core module as `List.head`. And make this function a `Task` too, as that will ease chaining things alltogether and allow us to expose an error message when the list is empty:

```haskell
pickFirst : List String -> Task String String
pickFirst events =
    case List.head events of
        Just event ->
            Task.succeed event

        Nothing ->
            Task.fail "No events."
```

So in order to chain all the pieces we have so far, we obviously need glue. And this glue is `Task.andThen`. Its signature is:

```haskell
andThen : (a -> Task x b) -> Task x a -> Task x b
```

The first argument, which is a function, will have to accept the result of a previous succesful task (the second argument), and return another task exposing a next value. The whole thing is a new Task exposing that new value. That means we can chain things this way, which is quite fancy:  

```haskell
fetchEvents
    |> Task.andThen pickFirst
    |> Task.andThen fetchName
```

Neat. But wait. How are we going to execute this macro-task? Here comes `Task.attempt`, which signature is:

```haskell
attempt : (Result x a -> msg) -> Task x a -> Cmd msg
```

Woah, where does this `Result` come from? And what is this `Cmd msg` thing?

- `Result x a` is the [Result] from the actual execution of our HTTP request, `x` being the error and `a` the succesfully received value
- `msg` here is a message, something we don't have defined yet; it's gonna be a value from a `Msg` type
- `Cmd msg` is the usual way in Elm to describe side effects  

So let's define a `Msg` type:

```haskell
type Msg
    = Name (Result String String)
```

`Name` conforms to the `(Result x a -> msg)` type, so we can use it directly with `Task.attempt`:

```haskell
fetchEvents
    |> Task.andThen pickFirst
    |> Task.andThen fetchName
    |> Task.attempt Name
```

But now, how are we going to run all this and do actual HTTP requests and process the responses accordingly? We need to setup the [Elm Architecture]:

```haskell
module Main exposing (main)

import Html exposing (..)
import Http
import Json.Decode as Decode
import Task exposing (Task)


type alias Model =
    { name : Maybe String
    , error : String
    }

type Msg
    = Name (Result String String)

eventsRequest : Http.Request (List String)
eventsRequest =
    Http.get "https://api.github.com/events"
        (Decode.at [ "actor", "login" ] Decode.string
            |> Decode.list
        )

nameRequest : String -> Http.Request String
nameRequest login =
    Http.get ("https://api.github.com/users/" ++ login)
        (Decode.at [ "name" ]
            (Decode.oneOf
                [ Decode.string
                , Decode.null "unspecified"
                ]
            )
        )

toHttpTask : Http.Request a -> Task String a
toHttpTask request =
    request
        |> Http.toTask
        |> Task.mapError toString

fetchEvents : Task String (List String)
fetchEvents =
    toHttpTask eventsRequest

fetchName : String -> Task String String
fetchName login =
    toHttpTask (nameRequest login)

pickFirst : List String -> Task String String
pickFirst events =
    case List.head events of
        Just event ->
            Task.succeed event

        Nothing ->
            Task.fail "No events."

init : ( Model, Cmd Msg )
init =
    { name = Nothing, error = "" }
        ! [ fetchEvents
                |> Task.andThen pickFirst
                |> Task.andThen fetchName
                |> Task.attempt Name
          ]

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        Name (Ok name) ->
            { model | name = Just name } ! []

        Name (Err error) ->
            { model | error = error } ! []

view : Model -> Html Msg
view model =
    div []
        [ if model.error /= "" then
            div []
                [ h4 [] [ text "Error encountered" ]
                , pre [] [ text model.error ]
                ]
          else
            text ""
        , p [] [ text <| Maybe.withDefault "" model.name ]
        ]

main =
    Html.program
        { init = init
        , update = update
        , subscriptions = always Sub.none
        , view = view
        }
```

That's for sure much more code than with the JavaScript example, but don't forget that the Elm version renders HTML, not just logs in the console ;)

As always, an [Ellie](https://ellie-app.com/7Q9svdqRGa1/1) is publicly available so you can play around with the code.

[Elm Architecture]: https://guide.elm-lang.org/architecture/
[Http package]: http://package.elm-lang.org/packages/elm-lang/http/latest/Http
[Result]: http://package.elm-lang.org/packages/elm-lang/core/latest/Result
[Task]: http://package.elm-lang.org/packages/elm-lang/core/latest/Task
