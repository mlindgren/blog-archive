---
layout: page
title: "ORTS UI Customization with Lua"
date: 2015-01-05 23:00
comments: true
sharing: true
footer: true
---

_[The following is a report for a project I did for the University of Alberta's
[CMPUT 350](https://www.cs.ualberta.ca/undergraduate-students/course-directory/advanced-game-programming)
course in the Fall 2011 semester. For this project, I modified the
[Open Real Time Strategy (ORTS)](https://skatgame.net/mburo/orts/) game engine
to integrate a Lua virtual machine, allowing scripting of user interface
elements through Lua code. The source code for this project is not available.]_

## Goal
The goal of this project was to modify the Open Real-Time Strategy game engine
to allow for user interface (UI) customization through the use of the
[Lua](http://www.lua.org) scripting language.

## Prior Work
### World of Warcraft

The project itself was inspired by the user interface customization options
available in _World of Warcraft_, hereafter referred to as _WoW_. Numerous game
engines, including many predating _WoW_, provide scripting support in various
ways. In most cases, though, these game engines used scripting only as a way to
make development or game modification easier. By contrast, _WoW_ provides
user-centric scripting, encouraging users to customize their own game experience
through Lua scripts. _WoW_ was likely not the first game to include this
mechanic, but it was certainly instrumental in bringing the mechanic to
prominence; today, a scriptable user interface is a basic expectation for
Massively Multiplayer Online games (MMOs).

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/wow1.jpg" alt="A stock World of Warcraft user interface"/>
<br />
<small>
Figure 1: A (mostly) "stock" World of Warcraft user interface. Copyright
unknown. Used under Fair Use.
</small>
</div><br />

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/wow2.jpg" alt="A heavily modified World of Warcraft user interface"/>
<br />
<small>
Figure 2: A heavily modified World of Warcraft user interface. Copyright
unknown. Used under Fair Use.
</small>
</div><br />

In the above figures, the contrast between the "stock" _World of Warcraft_ user
interface is and the modified interface is readily apparent. In addition to the
obvious visual differences, experienced _WoW_ players will notice that the
player in the lower screenshot is using mods which provide him or her with extra
information not easily attainable from the stock UI. For instance, the
[Grid](http://www.wowinterface.com/downloads/info5747-Grid.html) UI mod
(lower right) shows important status information about other members of the
player's raid group which the stock UI cannot show in a convenient manner.
Although Blizzard Entertainment, the developer of WoW, has balanced scripting
capabilities to ensure that they do not provide an unfair gameplay
advantage[^advantage], many players find it hard to play without the extra
information and input options provided by UI mods. Especially among highly
skilled and competitive players, certain modifications are
indispensable&mdash;again, Grid is a good example.

The success of user interface scripting in _WoW_ is such that, as previously
mentioned, it is now considered a standard feature of MMOs and is expected to be
included at launch.  In 2011, developer Trion Worlds released the critically
acclaimed MMO _Rift_ without scripting support, but with a much more
customizable stock interface than _WoW_ has to this day. Nevertheless, after a
huge demand from players, Trion did eventually add user interface scripting,
also using Lua.

### ORTS Scripting

Unbeknownst to me at the outset of the project, ORTS itself provides a robust
scripting system for both gameplay and user interface scripting. The system uses
a completely custom language designed and implemented by one of the project's
original developers. While it is impressive that the small development team
designed and implemented a custom scripting environment, I feel that there are
some significant shortcomings to his approach when considered from the
perspective of a general game development project:

- Whereas there exists a large community of experienced Lua developers,
  including many who have prior experience with user interface scripting for
  video games, there are obviously no developers familiar with any custom
  "in-house" scripting system. This limits the accessibility of the scripting
  features.
- In addition to the above, a custom language means that the development team
  themselves will be unable to leverage existing skillsets.
- Developing and implementing a custom scripting system is a significant
  undertaking and potentially a source of many difficult-to-fix bugs.
- A custom scripting engine which is even moderately generalized is unlikely to
  be as efficient as a standalone scripting system.

For these reasons, I believe that there is a trend in the industry to move
towards established languages for in-game scripting support. Thus, although my
project is something of a retread, it is not without merit, as it provides an
opportunity to examine what I feel is a more modern approach to in-game
scripting.

## Process

At a broad level, I undertook the following four steps in implementing this
project:

1. Integrate the Lua runtime environment into the ORTS codebase.
1. Expose new and existing ORTS functions to the Lua runtime, allowing it to
   query and in limited ways modify game state.
1. Create Lua hooks for relevant game events such as frame redraw, allowing the
   Lua runtime to respond to such events.
1. Design and implement a framework for the user interface in Lua, as well as
   sample UIs.

### Step One: Lua Integration
The first step was also the easiest, as Lua was designed to be easily
embeddable. Integrating it with ORTS was a simple manner of downloading the Lua
source code, putting it in a convenient directory, and integrating it with the
ORTS makefiles. From there, any ORTS code needing access to Lua needed only to
call a few simple C functions to set up a Lua state and start executing code.

The most difficult part of this step was, surprisingly, finding a way to get Lua
to build with ORTS. (It was not excessively difficult, just more difficult than
the other parts.) I wanted the ORTS makefile to build Lua rather than requiring
the user to manually install it on his or her system. However, ORTS Makefile
expects a certain structure in the files that it builds, and this makes it
poorly suited for integration with third-party libraries which have their own
specific build requirements. In the end, I simply added a rule to the ORTS
makefile which explicitly called make on the Lua makefile. This is a
less-than-ideal solution since it circumvents the typical ORTS library build
process in a sense, but there was no easier way to go about it without
significant overall changes.

### Step Two: Exposing ORTS Functions to Lua

This step was perhaps the most challenging as it required me to develop an
in-depth understanding of the system for calling C functions from within Lua.
The Lua runtime uses a register-based virtual machine which operates in a manner
that should be familiar to anyone with assembly language programming experience.
Although the virtual machine is register-based, when interacting with Lua from C
one is only concerned with the stack. The typical process for calling a C
function from Lua is as follows:

1. Create a Lua wrapper for the function you wish to call. The wrapper must have
   a particular signature in order to be registrable with the Lua runtime.
1. Within the wrapper, check the size of the Lua virtual machine (VM) stack to
   determine how many arguments were given. (If an unexpected number of arguments
   were present, the developer will have to determine how to handle the error.)
1. Using functions provided by the Lua API, retrieve argument values from the
   Lua stack. The developer should check the arguments to ensure that they are
   valid for the C function to be called.
1. Call the C function with the retrieved arguments.
1. Report any errors and return the result to the Lua runtime by placing it on
   the stack.

The process is not complicated, but there are a number of important caveats.
First of all, the astute reader will notice that I've been referring to C rather
than C++. Lua is written in C, and provides only a C API. This results in some
challenges when interfacing with it using C++:

- Functions exposed to Lua cannot be member functions. If you need to call a
  member function, you have to wrap it in a non-member function, and that
  function must have access to the object to call the member function on in
  one way or another.
- Lua provides types for passing, retrieving and even interacting with C
  structures in the Lua VM, using pointers. However, as soon as you pass a
  pointer to Lua, type safety is lost.
- Additionally, memory management is a concern with pointers passed to Lua.
  Although the Lua VM cannot deallocate your C/C++ objects, you must take care
  to ensure they are not deallocated while the Lua VM might be using them.
- Like many other scripting languages, Lua's introspection features make it hard
  to "hide" data from the user. This means the user could modify any pointer
  passed to the Lua VM, which could result in crashes or exploits.
- It is easy to write unsafe C/C++ code which can be called from the Lua VM and
  cause the host application to crash.
- Additionally, if one is not careful to properly use protected calls when
  calling from C/C++ to Lua, errors in the Lua code can crash the host
  application.

Due to the limited time available for this project, I was not able to come up
with a robust solution to the pointer modification problem; thus, in my ORTS
code, there are certain pointers exposed in the Lua VM which an astute user
could theoretically modify to exploit the game. (That said, the somewhat
atypical client-server architecture of ORTS probably severely limits any
advantages the user could hope to give himself. A more likely scenario is that
an unwitting user could accidentally modify the pointers and cause the
application to crash.)

In the interest of rapid development, my goal was to provide as few ORTS
functions to the Lua VM as possible, and to write all of the user interface code
in Lua. I was largely successful in this endeavor, and in a few cases I managed
to take advantage of some of ORTS' existing UI scripting functions.

One of the most challenging aspects of exposing ORTS functionality and game
information to Lua came in the form of making my scripting system interact with
the existing scripting system. In ORTS, game rules and unit attributes are
defined in scripts. Thus, for my Lua scripts to be able to access game state, I
had to figure out how to call the relevant functions which were designed for the
original scripting system.

For some aspects of development this proved to be a minor inconvenience, whereas
for others it became a major stumbling block. For instance, the original
scripting system is by necessity tightly integrated with the "action" system for
in game units. Each unit defines its actions and the buttons corresponding to
those actions entirely in scripts. The complex manner in which these actions are
defined and organized meant that I had to fall back on the existing UI for
actions and action buttons, as I did not have time to reverse engineer all of
the existing scripts and rewrite them in Lua.

Needless to say, integrating the two scripting systems was a complicated process
and resulted in some less-than-optimal code. If one were to pursue the idea of
using Lua scripting in ORTS, ideally the original scripting system would be
replaced and all gameplay information would be defined in Lua scripts.
Naturally, that would make it much easier for the user interface to interact
with game state. That said, replacing the original scripting system entirely
with Lua would probably be tantamount to a complete rewrite, given how the tight
integration between the scripting system and other game systems. This could be a
potential consideration for the next major iteration of the ORTS project.

### Step Three: Notify Lua of Game Events

This step was relatively simple. Calling Lua code from C/C++ is a simple
matter of using the Lua API to find the "index" of the relevant Lua function,
pushing the arguments onto the stack, and then executing the function. Thus, I
was able to notify Lua (by calling user-defined Lua functions) of the following
events:

- During frame redraw, Lua is notified after the 3D world is drawn. This allows
Lua to draw the user interface on top of 3D world.
- Lua is notified when the mouse is moved or a mouse button is pressed.
- Lua is notified when a key is pressed.

### Step Four: Lua UI Framework
For this step, I leveraged my experience in designing and implementing UI
components for my iPhone game, [Puzzle Panel](/projects/puzzle-panel/). For
simplicity, the UI framework uses a simple delegate model so that individual
"widgets" can respond to events.

The user creates a layout by adding widgets to the framework using a provided
function. There is no pre-defined structure for a widget; it can be any Lua
class[^class], so long as it provides all of the methods necessary to respond to
actions for which it has been marked a delegate. Naturally, all widgets are
drawable, so each time the framework is notified by ORTS of a frame redraw, the
draw method of each widget is called. Widgets are drawn in the order they were
added to the scene.

In addition to drawing, widgets can be registered as key listeners or mouse
listeners. Key listeners are notified of key presses by the framework; mouse
listeners are notified of mouse presses and movement. The notification occurs in
order of priorities specified by the user, and any listener can choose to
"consume" relevant events by returning `true` in its notification function.
Consumed events are not propagated to lower-priority listeners, and
additionally, are not passed on to the native ORTS code. Thus, Lua scripts can
override default ORTS behavior. (Mouse movement cannot be consumed, because
otherwise the if any widget consumed mouse movement the user would no longer be
able to interact with the game.)

Using the simple primitives I set up in steps two and three, I was able to build
a fairly complex user interface system which could, with enough additional Lua
code, provide most of the functionality expected of modern windowing systems.
Although the time I had available for this project limited the number of widgets
I was able to implement, many more widgets are technically possible.

### Challenges Encountered

In addition to the challenges already discussed in the Process section, there
were a number of general challenges which I had to overcome in undertaking this
project. First and foremost was simply learning to work with ORTS itself.
Although it is an impressive research project, ORTS has not seen serious
development in a number of years, and it has a number of serious issues which
impede further development efforts.

- A large number of dependencies must be manually installed to compile and run
  ORTS (if one is working in an environment where those dependencies have not
  already been installed, i.e. other than the University of Alberta lab
  machines.)
- Although it claims to be platform-independent, getting ORTS to build on modern
  Mac OS X is very difficult. Even after successfully compiling it, I was never
  able to get it to run properly; the `ortsg` window never appears, and both
  orts and `ortsg` crash on exit.  I spent a great deal of time trying to find
  workarounds for these problems, but to no avail. I think it is likely that
  were they overcome, other problems would occur on Mac OS X as well. In
  addition, I have heard from other students that `ortsg` freezes after a matter
  of seconds when built and executed on Windows.
- The _unmodified_ `ortsg` application suffers from seemingly random (and
  therefore not easily reproducible, and hence not easily debuggable)
  segmentation faults on the lab machines. Segmentation faults occasionally
  occur on startup, but more often at random times during gameplay. Obviously,
  it was difficult to guarantee that my own code did not cause segmentation
  faults when the existing code already did at random.
- Many or perhaps all of the example games and applications included with the
  ORTS source code do not seem to build and run properly&mdash;at least, not
  without invoking some black magic compilation procedure which is not made
  clear by the sparse documentation.

Compilation and segmentation fault issues notwithstanding, the ORTS code is not
very easy to work with in some respects. The code suffers from a serious lack of
comments in many places, which meant that figuring out which functions to call
and where was largely a process of trial and error for me.

The extent to which the code is modularized and the particular manner in which
it is organized makes it difficult to quickly gain an understanding of. This
problem is exacerbated by the occasional use of complicated preprocessor macros.
To some extent I suspect these problems are endemic to large projects in
general, and with time the structure of the ORTS system as a whole would have
likely become more familiar and made more sense to me.

In the process section, I discussed in detail the challenges I encountered in
integrating Lua with the ORTS C++ code. Most of these challenges arose due to
the fact that the Lua API is a C API. There are frameworks designed to ease the
process of integrating Lua with C++; one such framework is
[Luabind](http://www.rasterbar.com/products/luabind.html). It likely
would have been beneficial in this project. Unfortunately, Luabind requires the
[Boost.Jam](http://www.boost.org/build/doc/html/bbv2/jam.html) build system.
After some investigation, I decided that setting up Boost.Jam and, in
particular, integrating it with the existing ORTS build process would take up
too much time that I could otherwise spend on functionality, and decided to
hand-write all of my Lua integration code. Of course, one advantage to not using
Luabind is that I gained a greater understanding of the standard Lua API than I
would have had I used it.

## Results

The following series of screenshots serve as an example of the basic
functionality that can be accomplished with my Lua UI modification system:

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/orts1.png" alt="The standard ortsg user interface."/>
<br />
<small>
Figure 3: The standard <code>ortsg</code> user interface.
</small>
</div><br />

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/orts2.png" alt="An ortsg interface modified with Lua, mimicking StarCraft 2."/>
<br />
<small>
Figure 4: An <code>ortsg</code> interface modified with Lua, mimicking <em>StarCraft 2</em>.
</small>
</div><br />

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/orts3.png" alt="A minimalist orstg interface."/>
<br />
<small>
Figure 5: A minimalist <code>ortsg</code> interface.
</small>
</div><br />

While the changes may appear primarily cosmetic, a significant amount of work is
taking place on the backend to relay information from the standard ORTS
scripting environment to the Lua scripting environment. For instance, in order
to simply draw the player's resource counts, the Lua script must be able to
query the game state, find the scripted player object, access its resource count
attributes by name, and retrieve the values. All of this is non-trivial.

## Possible Improvements
- Scripted actions from the existing system could be exposed to Lua, which would
  allow for the interface to be design entirely in Lua. As mentioned in the
  process section, currently the Lua interface falls back on the pre-existing
  scripted interface for actions and action buttons due to complications in
  interfacing the two. This would be a significant undertaking.
- A more robust way to provide access to C++ objects from Lua would be
  beneficial from a security and stability perspective. As previously mentioned,
  currently raw C++ pointers are exposed to Lua, which negates type safety and
  allows script-writers to perform unsafe operations.

  I suspect that the proper way to handle this is to give Lua access only to
  some sort of non-pointer identifier for objects it needs to know about. For
  instance, for C++ objects stored in a `std::map<string, ?>`, Lua could access
  the objects by their identifying strings. Unfortunately this requires an
  overall system architecture designed with special Lua identifiers in mind,
  which made it unfeasible for this project. An alternative would have been to
  maintain a global table mapping some unique ID to a smart pointer to an
  object, for each object that should be accessible by Lua. Unfortunately this
  too was not feasible to implement within the project timeline.
- Currently modifications such as adding and removing widgets from the UI must
  be performed in Lua code, or in the Lua console widget (by executing Lua
  code.) It would be much more user-friendly to provide and in-game interface
  for adding, enabling and disabling UI scripts.

## Conclusion

In the project proposal, I set out three metrics on which to determine the
success or failure of the project. The metrics were as follows:

#### 1. "ORTS users are able to customize their user interfaces through the use of Lua scripts."

Obviously the project has been successful in this regard. I have successfully
integrated Lua and provided the means to build a completely custom UI. As
previously mentioned, Lua provides the advantage of being a widely-used
scripting language, so whereas there would be a significant learning curve to
creating UI scripts the custom language that the project used previously, anyone
with UI scripting experience in any of the triple-A games using Lua will be
comfortable with my framework.

#### 2. "The customization options are non-trivial."

In this regard I must confess that I did not accomplish as much as I had hoped
to be able to, due to limited time and a heavy course load. I had to focus on
reproducing standard UI functionality in my framework, and although the
resulting system offers potentially unlimited cosmetic options, the ability to
interact with game state from within Lua scripts is limited.

Ideally, I would have liked to implement features allowing limited unit control
via scripting. Although micromanagement of units is an important measure of
skill for top-tier RTS players, as a player of (extremely) limited skill I've
always found it tedious to the point of interfering with my enjoyment of the
games. To that end, I was planning to give players the option to write simple
"AI scripts" which could be executed on command to make units perform certain
actions; this would have provided a means of responding to varied situations at
the "micro" level without requiring a high rate of actions-per-minute (APM).
Unfortunately, I had to cut this feature due to time limitations.

Nevertheless, my framework does offer players the opportunity to display game
information in whatever layout and format best suits their needs. Another
problem I have with many RTS games is that the UIs tend to be large and
cumbersome, obscuring a significant portion of the screen. Even without any of
the functionality I described above as missing, my framework would certainly
solve that properly when applied to a typical RTS.

#### 3. "Customization is simple enough that anyone with a programming background can pick it up easily."

This is one area where I feel that my framework excels, particularly in
comparison to the pre-existing ORTS UI scripting system. Lua has more than a few
syntactic and design quirks, but for the most part it is similar enough to the
most popular scripting languages (e.g. PHP, Python, Perl and Ruby) that anyone
experienced in one of those languages should be able to grasp it very quickly.
Indeed, prior to embarking on this project I had no experience whatsoever with
Lua.

By contrast, the pre-existing scripting language has a more verbose C-style
syntax which is mixes in unusual syntax such as variable declarations which are
not delimited by semicolons as other statements are. Semicolons seem to take the
place of commas in some function calls but not others, and there is a
distinction between "classes" and "blueprints" which is anything but intuitive.
As I mentioned previously, I think that this custom scripting system is an
impressive technical achievement, and I do not wish to disparage the work that
was put into it. I merely highlight these peculiarities to reinforce my earlier
assertion that an established scripting language is preferable to a custom
language. Documentation alone makes it much easier for a new user to get up to
speed with Lua than with ORTS' pre-existing scripting language, as documentation
for the latter is extremely limited.

Besides the contrast between the languages themselves, I designed the framework
in such a manner as to make interactions and patterns simple and obvious. The
delegate model is used in a number of real-world user interface libraries
including Apple's Cocoa library, and although mine is obviously less polished,
the presence of familiar conventions should make it easy for inexperienced users
to add new widgets.

## Conclusion

On the whole I am satisfied with the outcome of the project. I have gained
valuable experience in designing user-oriented scripting systems, and I am now
aware of some of the challenges involved in providing client-side scripting in
games. I now feel confident that I will be able to successfully integrate Lua
scripting into my personal projects in the future, which will no doubt be
beneficial in a number of scenarios.

[^advantage]: Despite Blizzard's best efforts, user interface customizations have caused balance issues in the past. For instance, early in WoW's history, a script called Cursive trivialized some boss fights by providing near- automatic removal of negative spell effects on party members. Obviously, any game developer providing public scripting capabilities must be wary of situations such as these.

[^class]: Strictly speaking, Lua is not object-oriented, but its built in structures provide a close approximation of object orientation. For simplicity, I use the world "class." The specifics of mimicking object orientation in Lua are surprisingly complicated.
