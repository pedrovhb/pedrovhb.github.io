---
title: "Quick proof of concept - async GUI with DearPyGui"
date: 2021-07-09T10:48:12-03:00
draft: true
---

I start a lot of idea explorations that have an interesting core, but I sometimes just don't finish pursuing.
I just conjured up this one which I think has some potential, and since posting on my blog is free, here it is :)

Since I've been diving into asyncio and especially async iterables lately, the habit of looking at things through an async stream viewpoint is fresh in my mind.
Applying the proverbial hammer to GUIs, I could envision a pretty neat style for programming GUIs which I'd like more than what I've seen so far.

DearPyGui is a Python library based on the Dear ImGui library, which is an immediate mode GUI library. 
That means that state management is a kind of declarative up in the air free-for-all style thing. 
Clearly I have no idea what I'm talking about, and this exploration is either something clever or just a reinvention of the problems immediate mode GUIs were designed to solve in the first place (software development is a circle and it's hard to keep track of where we're at, at this point), but my thought was that the whole thing was refreshingly simple and I was ready to corrupt that.

Anyway, [here's some draft code I came up with](https://gist.github.com/pedrovhb/2a1ebaf65c0ceaf317bb88400db04c3d) to show how I think it could work.
That link shows the full file, but to be brief here's the main section which demonstrates how the idea works in terms of the interface:

<script src="https://gist.github.com/pedrovhb/2a1ebaf65c0ceaf317bb88400db04c3d.js?file=dearasyncgui_fragment_1-py"></script>

It seems to me that asyncio provides a great way to do things which would otherwise be fairly inelegant to do.
Of course, it's unlikely you'd do something like that in your main function, but you can easily and efficiently set up various async tasks to wait on different things and update the GUI accordingly without getting lost in creepy callback catacombs, tattered thread tangles, or shared state spaghetti.

Do note that this is scratch code. 
I'm trying to be less perfectionist before sharing things (which just ends up meaning I don't share things).
In particular, it's fairly repetitive in the implementation of the different event and component classes, but I like to start things out this way so that common ideas and differences are more obvious.

A few things to note:

- Buttons are clicked, sliders are slode (naturally)
- There's pretty clearly a repetitive pattern in the shape of events and components, so some not-too-complex abstraction could possibly handle most everything.
- 


And here's a secret - the internal implementation *is actually really simple*.
It's great to use complicated words and jargon to make ourselves sound smart (I do it liberally myself), and asyncio is a hard to grasp beast at times, but here: I'll post the implementation to the Slider class, and you'll understand it without any comments and without having to scroll your screen:

<script src="https://gist.github.com/pedrovhb/2a1ebaf65c0ceaf317bb88400db04c3d.js?file=dearasyncgui_fragment_2-py"></script>

Look at us, expertly utilizing futures to power our asynchronous iterators in the event loop like it's not even a thing.


