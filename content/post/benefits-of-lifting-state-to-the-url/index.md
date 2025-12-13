+++
title = 'Benefits of Lifting State to the URL'
date = 2024-06-27T18:45:02-04:00
draft = false
tags = ["State Management"]
+++

I recently found myself needing to lift some parts of my application state into the URL.
I usually default to use some state management solution in my web projects, and I am
a huge fan of [Zustand](https://spin.atomicobject.com/zustand-alternative-redux/).
However, there comes a time when using this type of state management alone may
not provide all the desired functionality.

## Why Lift State

Normally, when using something like Redux, the global store lives in memory within
the JavaScript runtime of your application. This generally would work great, but what if
a user wanted to share part of their application state with another user running the
application on another computer? Or what if the same user simply wanted to share some
application state between different tabs in the same browser? If all your state
is stored in runtime memory, this isn't possible.

The solution: put the part of the state that you want to share between instances into
the URL. For example, on the [PhotoVoice Japan](https://photovoicejapan.com) website, I wanted
users to easily be able to share voices. Initially, a user would have to go to the "search"
tab, explain what search params to use and then also share the title of the voice that
they wanted the other user to view. But after lifting the voice ID (in my case I am using Contentful's
internal Entry ID, but this can be any unique identifier) into the URL, they need
only share a single URL to link directly to a specific voice. For example: https://photovoicejapan.com/display/23z0n4kUPKvV0KYDbwTtAz.

This change not only added a new feature in being able to share specific voices, but also
simplified the existing Zustand store, by removing the concept of a "currentVoice" and
simply having that "currentVoice" entry ID live in the URL. Furthermore, I find having
this part of the state in the URL can aid in debugging, since "reproducing" state
is much easier by simply sharing a URL, instead of needing to click through multiple UI
windows/modals.

## Time to Put Everything in the URL?

Given the clear benefits of lifting state to the URL, why not just move everything there?
For some things, it probably just isn't important enough to move to the URL or
serves no real purpose. For example, the user's light/dark mode preference. This can just live
in memory and default to the user's browser preference. If it's very important that this
preference is saved, it can be linked somehow to a concept of a user and their "profile."
Additionally, some pieces of state do not make sense to share. For instance, maybe the user
is in the middle of entering data or making selections in a multi-step modal. It is likely
that such a workflow would be all or nothing, and it is therefore unnecessary to store the
intermediate parts of state in the URL.

## Enjoying the Benefits Multilocation State

Lifting state to the URL is helpful when needing to easily share state between users or
instances and to aid in state reproduction for debugging and troubleshooting purposes.
Utilizing both runtime memory and the URL to manage state is important as applications
become more complex. I hope you find that lifting state to the URL can also help improve
the functionality of your applications!
