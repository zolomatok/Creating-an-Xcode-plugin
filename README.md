# Creating-an-Xcode-plugin
Xcode is awesome. No doubt.  Sure, it crashes at times, frustrates us with code signing and there are certainly missing features and minor tweeks that would make it even greater.

There is a nice list of available plugins over at [NSHipster](http://nshipster.com/xcode-plugins/) that aim to help with a lot of theese.

Now here is the fun part! You, yes YOU can develop your own plugin and make Xcode do **anything**!

<p align="center"><img src="images/xcp-tut-face-motherofgod.png" width="400" border="1"/></p>

Remember, there is no app review process on OS X (outside of the Mac App Store) which means we can use private frameworks! 

Unfortunately there is also no official documentation on creating plugins. Members of the community, however, have scouted some of the way into Pluginland making our joruney a tad easier.

Creating an Xcore plugin feels like an adventure on an uncharted treasure island. It is a challenging trip and a great learning experience. You will never look at Xcode the same way ever again.

This tutorial will teach you how to **get started**, what **tools** to use and what **workflow** to follow when developing an Xcode plugin. 
We will create a plugin that lists the document items (interfaces, properties, methods, etc..) in the right side panel. Clicking an item will select it in the source code window and scroll to its position. It’s basically a watered down version of [ZMDocItemInspector](https://github.com/zolomatok/ZMDocItemInspector).

**Note:** This tutorial shamelessly rips off the style of those found on [raywenderlich.com](http://www.raywenderlich.com/), since they work so well. That means face memes, everybody.

