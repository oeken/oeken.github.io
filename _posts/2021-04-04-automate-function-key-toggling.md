---
layout: post
title:  "Automate toggling function keys in macOS" 
date:   2021-04-04
description: Learn how to swiftly switch between standard and system function keys.
comments: true
permalink: automate-toggling-fn-key
---

This one is not a coding problem but a productivity tip. I mostly use JetBrain IDEs for programming on macOS. I need frequent access to function keys (`F7`, `F8`, `F9`, etc. ) when I debug some code. Not only those, but also I need access to system function keys very often, to adjust brightness, volume, and to get an overview of open applications, etc.

In this case, being able to quickly toggle the mode of the function keys would be very useful:
* Standard function keys (`F1`, `F2`, etc.)
* System function keys (Adjust brightness, volume etc.)

It is possible to do this in System Preferences --> Keyboard, however, it is very slow having to go through these panels.

{:refdef: style="text-align: center;"}
![Keyboard Preferences](/assets/automate-toggling-fn-key/keyboard-preferences-annotated.png)
{:refdef}

Using **Automator** and **BetterTouchTool** programs we can do much better and faster. In Automator, one can write custom scripts in **AppleScript** language to control the system. For example, the following snippet toggles a checkbox in the **Keyboard Preferences** on macOS Big Sur (11.2.2), coincidentally... exactly the one we need.

<script src="https://gist.github.com/oeken/86e1dd95b7f8e8cdc95f41d74bc9e092.js"></script>

Next, create a **macOS workflow** with this script and save it where appropriate.

{:refdef: style="text-align: center;"}
![Keyboard Preferences](/assets/automate-toggling-fn-key/automator-annotated.png)
{:refdef}
> Automator --> File --> New --> Worfklow --> Run AppleScript.

Verify that workflow does toggle the function keys by executing the workflow. Next, bind this workflow to a keyboard shortcut using BetterTouchTool.

{:refdef: style="text-align: center;"}
![Keyboard Preferences](/assets/automate-toggling-fn-key/better-touch-tool-annotated.png)
{:refdef}

In my case, instead of a key combination, I chose to bind it to a key sequence: _Fn button pressed twice consecutively_.

And voilà!

Now, when programming I can switch to standard mode to use keys `F7`, `F8`, `F9` and when I want to control my computer I can switch back to system function keys.

<center>
    <figure style="display:inline-block;">
        <img width="80%" src="/assets/automate-toggling-fn-key/clion.png" style="margin-top:20px;margin-bottom:30px;margin-right:10px">
    </figure>
</center>
