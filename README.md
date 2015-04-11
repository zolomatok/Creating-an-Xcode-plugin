# Creating an Xcode plugin
Xcode is awesome. No doubt.  Sure, it crashes at times, frustrates us with code signing and there are certainly missing features and minor tweeks that would make it even greater.

There is a nice list of available plugins over at [NSHipster](http://nshipster.com/xcode-plugins/) that aim to help with a lot of theese.

Now here is the fun part! You, yes **YOU** can develop your own plugin and make Xcode do **anything**!

<p align="center"><img src="images/xcp-tut-face-motherofgod.png" width="400" border="1"/></p>

Remember, there is no app review process on OS X (outside of the Mac App Store) which means we can use private frameworks! 

Unfortunately there is also no official documentation on creating plugins. Members of the community, however, have scouted some of the way into Pluginland making our joruney a tad easier.

Creating an Xcore plugin feels like an adventure on an uncharted treasure island. It is a challenging trip and a great learning experience. You will never look at Xcode the same way ever again.

This tutorial will teach you how to **get started**, what **tools** to use and what **workflow** to follow when developing an Xcode plugin. 
We will create a plugin that lists the document items (interfaces, properties, methods, etc..) in the right side panel. Clicking an item will select it in the source code window and scroll to its position. It’s basically a watered down version of [ZMDocItemInspector](https://github.com/zolomatok/ZMDocItemInspector).

**Note:** This tutorial shamelessly rips off the style of those found on [raywenderlich.com](http://www.raywenderlich.com/), since they work so well. That means face memes, everybody.

## Workflow
The hardest part of the process is to find out where to begin. How do we even start developing a plugin? Lack of documentation aside, Xcode actually provides great support for plugins and internally it makes use of this feature heavily.

In order to have a greater understanding of the whole process, let me introduce you to the general **workflow**:

1. **Create a project with a plugin template** and use the initializer of the plugin as the point of insertion
2. **Browse the Xcode runtime headers** for clues on what to hook into. These headers contain the names of the protocols, properties and methods of every single class Xcode relies on
3. **Try and intercept method calls** in the target classes to replace their implementation with our own
4. **Look at the notifications Xcode broadcasts** to find out if there are broadcast events we are interested in

## Toolkit
Fortunately we can find tools for each of these tasks that will make our lives infinitly more pleasant. Time to assemble our **toolkit**!

1. Download and install **[Alcatraz](http://alcatraz.io/)**! It’s a plugin manager for Xcode which willl let us download and install the necessary plugins with the click of a button.
2. Get the following plugins via Alcatraz by going to **Window -> Package Manager** in Xcode.
 2. **“Xcode Plugin”** (on the **Templates** tab)
 2. **“XcodeExplorer”**
3. Download the **[Xcode Runtime Headers](https://github.com/zolomatok/Xcode6-RuntimeHeaders)** from GitHub.
Note, that these headers are not the real class headers. They contain every single property and method found in the original implementation files too (not just the header files)! These headers are generated by running the super fantastic class-dump command line tool on all the libraries found in /Applications/Xcode.app.
4. Get **[DTXcodeUtils](https://github.com/thurn/DTXcodeUtils)** either by downloading it from GitHub or using CocoaPods. DTXcodeUtils are just a few header files for common operations, like inspecting the current source code editor.
5. Lastly, download **[JGMethodSwizzler](https://github.com/JonasGessner/JGMethodSwizzler)**. More on method swizzling later, but this is where the magic happens.

Looking at the Xcode header files, you might notice that it is around **12,000** classes and protocols.

<p align="center"><img src="images/xcp-tut-pokerface.png" width="400" border="1"/></p>

**NOTE: There seems to be a slight incompatibility between Xcode Explorer and Alcatraz.** In Xcode open the **Window** menu and see if you find the **Explorer** menu item. If it is there, no problem. If not, you can fix it the following way:

1. Download the **[Xcode Explorer](https://github.com/edwardaux/XcodeExplorer)** source from GitHub.
2. In **XCEPlugin.m** find the line *NSInteger organizerMenuItem = [[menuItem submenu] indexOfItemWithTitle:@“Organizer"];*
3. Change **@“Organizer”** to **@“Devices”**
4. Hit **build**
5. Restart Xcode
6. Voila!

***This tutorial is quite a read. To keep things organized, the rest of the article will follow the steps of the workflow described above.***

## 1: The Project
Start up Xcode 6, and go to **File \ New \ Project…** Select **Xcode Plugin**, name it **AwesomePlugin**.
<p align="center"><img src="images/xcp-tut-create.png" width="600" border="1"/></p>

This will create a new project with a single class which has the **name of your project**.
Let us take note of what we see in **AwesomePlugin.m** right off the bat:

1. There is a **sharedPlugin singleton**. This will come in handy during the method swizzling.
2. **-initWithBundle:** seems to be our point of insertion, and indeed there is already some code there regarding the menu

*For now, hit* ***build and run*** *and see what happens!*
<p align="center"><img src="images/xcp-tut-sorcery.jpg" width="400" border="1"/></p>

That’s right, a new Xcode instance was opened since Xcode can debug itself! We can use all the standard debugging tools too, including breakpoints!

- - -
*Now click the* ***Edit menu***, *then the ***Do Action*** *menu item, in the* ***newly opened Xcode instance***. Hello World!

You will find that the Do Action menu item is only available in the second Xcode. That’s because plugin loading happens when Xcode is launched. 

Since the plugin **binary is copied into the plugin directory** as part of the build process, if you quit Xcode completely and launch it again, the menu item will appear without building and running the project again.

The plugin directory for Xcode 6 is **~/Library/Application Support/Developer/Shared/Xcode/Plug-ins.** 

It is a good idea to create an **alias** for this folder somewhere that is convenient for you. If we intoruce a bug in the plugin that makes it crash (something that happens all the time) and than quit Xcode, it will keep crashing on launch time and we will not even be able to start Xcode again.

The way to solve this is by uninstalling the plugin by **deleting it from the plugin directory**, then restarting Xcode.

- - -
*Let’s setup our project.*

1. You can delete the **-doMenuAction** method, and the **“// Sample Menu Item:”** section of **-initWithBundle:**
2. Add **JGMethodSwizzler.h** and **JGMethodSwizzler.m** into your project by dragging them into the Project Navigator.
3. Add **DTXcodeHeaders.h**, **DTXcodeUtils.h** and **DTXcodeUtils.m** into the project also.

Additionally, if you are following this tutorial in the future (OMG HAX!) using **Xcode 7** or later you might need to add your Xcode version’s UUID into the **DVTPluginCompatibilityUUIDS** field in the Info.plist.

To read the UUID of your Xcode, paste the following snippet into **Terminal**:

`defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`

This will spit out the unique ID you need to copy into the Info.plist under DVTPluginCompatibilityUUIDS. This is to tell Xcode that your plugin is compatible with the current version.

<p align="center"><img src="images/xcp-tut-files.png" border="1"/></p>

