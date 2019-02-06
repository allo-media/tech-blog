---
layout: post
title:  "Elm-css Post Mortem"
excerpt: "Recently we started getting reports of huge perforances regression in our audio transcription player. We found out it was because of... styling!"
date:   2019-02-06 10:00:00 +0100
categories: elm
tags: elm elm-css css performance
---

Recently we started getting reports of huge performance regressions in our audio transcription player: the longer the audio was, the slower and sluggish the UI turned. (XXX big O?) The transcription UI is basically a table with one *sentence* per line, which itself is splitted into individual *words*. When playing the audio, the transcription UI highlights the matching sentence and the current word in it.

XXX capture peek gif

On a one hour audio, we can get up to 1200 sentences and/or 10000 words, depending on speech density. To sync all this up, that's a lot of DOM elements to list and update the further the audio plays!

We started studying common Elm DOM performance improvement techniques: [Keyed], [Lazy], [Zip lists], etc. but none really managed to solve the issue entirely: our UI was slow as hell, and for a brief moment in time we were even wondering if that wasn't a fundamental limitation of Elm or functional programming.

At some point though, we couldn't really believe Elm wasn't capable of dealing with a few hundreds DOM rerendering operations per seconds. Out of other ideas, we started inspecting seemingly unrelated stuff like the DOM nodes in the browser devtools inspector. And then, we saw this:

XXX capture multiple style lags

The hell? For every single table row we defined inline elm-css styles for, we had a single `<style>` tag. And the content was duplicated hundreds of times!

So what was really happening? When the audio plays, we listen to an `<audio>` node `timeupdate` event which is triggered a few times per second, notifying Elm subscribers of the current timecode the audio plays at, triggering Model updates and view renders accordingly. Elm-css adding a single `<style>` tag per row, and the browser having to parse all that and infer rendering accordingly a few thousand times per second put the browser on its knees.

## Conclusion

- we should have inspected our DOM earlier
- we should have leveraged performance devtools more

But we learned a lot

- ziplist
- lazy
- keyed
