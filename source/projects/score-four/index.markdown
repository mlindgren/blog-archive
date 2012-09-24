---
layout: page
title: "Score Four"
comments: true
sharing: true
footer: true
published: true
---

<a href="/images/scorefour_screen.png" title="Click here to view the full-sized screenshot."><img style="float: right;" title="Score Four screenshot" src="/images/scorefour_screen.png" width="280" height="200" alt="Score Four screenshot" /></a><p style="text-align: left;">Score Four was my term project when I took Computer Science 101 at the University of Northern British Columbia, taught by <a title="Dr. David Casperson's personal website" href="http://web.unbc.ca/~casper/" target="_blank">Dr. David Casperson</a>.  I worked on a team consisting of Erin Lutzer, Stephen Inwood, Yishan Hu, and myself.  The premise of the game is simple; it could be likened to three-dimensional tic-tac-toe.  On a four-by-four grid of pegs, your goal is to create a straight line of four beads of your color along any axis.  You can play against a computer opponent or takes turns with another human.</p>

<p style="text-align: left;">From a programming perspective, the interesting part of this project was the communication model specified between players in the game. To minimize coupling, the computer opponent communicates with the main game thread through a Java class which simulates a Unix pipe. The communication protocol was standardized for all teams in the class, so AI could be swapped out simply by replacing the appropriate class file. This also allowed AIs from various teams to compete against each other by means of a "tournament director" class. Additionally, it would have made network play very easy to implement, although doing so was beyond the scope of the project.</p>

<p>To ensure that our project stood out, I decided to challenge myself and create a 3D user interface using the Java3D API. This allowed me to make a simple game much more visually interesting than it otherwise might have been.</p>

<h2 style="text-align: left;">Get the game</h2>
<p style="text-align: left;">You can download the game from <a title="Scorefour game ZIP" href="http://www.mlindgren.ca/files/team3-scorefour.zip" target="_blank">here</a>.  To run it, simply extract both files to the same folder and double-click the JNLP file.  You'll need Java installed, of course.  If you don't have JNLP files associated with Java WebStart, you can start the game by running <code>javaws team3-scorefour.jnlp</code> in a terminal.</p>

<p><strong>The game does not run on Mac OS X</strong>. To the best of my knowledge this is caused by an incompatibility in the OS X Java3D binaries.  If you find a way around this, please leave a comment and let me know.</p>

<h2>How to play</h2>
<p style="text-align: left;">Click on a peg to place your bead. Use the left and right arrow keys to rotate the board.</p>
