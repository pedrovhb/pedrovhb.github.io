---
title: "Live snippets for JetBrains IDEs"
date: 2021-07-27T13:04:33-03:00 
draft: false
---

I love JetBrains IDEs. One thing they offer is customizable live snippets. These
are dynamic code templates which allow you to input common patterns quickly.
It's not just copy-pasting a set of templates; they can do things like position
the caret automatically at relevant points in the snippet and dynamically insert
things like the current file name or line number, or the date and time.

I'll try to keep this blog post updated as I create more live templates. I wish
we had a place for sharing these. There's probably lots of useful ways to use
this that I haven't thought of. Let me know if you have a nice one!

#### Timestamped todos

I write lots of temporary comments as I'm debugging, and I mark them with `todo`
so they stand out. It's often helpful to know if a note I made is from before or
after some change, so I have a live snippet that adds a todo with the time it
was added.

![Automatically timestamped todo comment](/img/2021/live_snippets/timestamped_todo.gif)

#### Silk profiling

This one also demonstrates how live templates can be helpful by interacting with
selections. **Protip**: if there isn't an active selection, invoking a surround
live template automatically selects the line at the caret minus leading
whitespace, which is often what you want.

Silk is a profiler for Django. You can decorate functions to be profiled or use
a context manager to profile code blocks. I made a live template for these:

![Silk decorator](/img/2021/live_snippets/silk_dec.gif)

![Silk context manager](/img/2021/live_snippets/silk_ctx_manager.gif)

#### JS - quick console log

This one does a `console.log` with the current file name and line. Unfortunately
these aren't automagically updated when things move around :)

![Console log](/img/2021/live_snippets/js_console_log.gif)

#### Python - unpack and print one item per line

When printing a Python list (or, more generally, a collection), items are
displayed all in the same line. Most times, I want to see each item in its own
line instead of squinting to look for a comma in a huge list that's been wrapped
a few times in the console.

![Print one per line](/img/2021/live_snippets/print_one_per_line.gif)

#### Using these ones

You can get these [here](/misc/2021/my_live_templates.zip). To add them to your
IDE, follow
[these instructions](https://www.jetbrains.com/help/pycharm/sharing-live-templates.html)
. Please do share any interesting ways you use live templates!
