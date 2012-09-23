---
layout: post
title: "How to link GLUT in Qt Creator on OS X"
date: 2011-09-21 00:34
comments: true
external-url:
categories: []
published: true
---
This tutorial is for an extremely specific and possibly unusual set of circumstances, but I'm hoping that it will prove useful to someone. For one of my courses this term, we're building OpenGL apps using Nokia's open source Qt framework. Qt provides some very nice scaffolding, and the Qt Creator IDE is among the best I've used, although I imagine it lacks flexibility for general-purpose applications—that is, you probably wouldn't use it to build non-Qt applications. Qt also allows one to easily write cross-platform C++ GUI applications. That said, linking procedures can be a bit different on different platforms, and as a beginner it's not always immediately obvious how instructions for Windows development translate into other environments.

<!--more-->

In particular, I found myself having difficulty linking the OpenGL Utility Toolkit (GLUT) with my project on Mac OS X. After much Googling and some frustration, I came up with the correct way to go about it, which is as follows:
<ol>
	<li>In Qt Creator, open your project file (.pro) in the text editor.  Right click on the text editor pane and select "Add Library..."</li>
	<li>Chose "System Library."</li>
	<li>Locate the GLUT framework.  On my Mac, running Lion, I used <code>/Developer/SDKs/MacOSX10.7.sdk/System/Library/Frameworks/GLUT.framework</code>. However, it seems there are multiple different copies on the disk that you could use.  If that path doesn't work for you, just search for <code>glut.h</code> in Finder and you'll find a copy of the framework.  The path you enter into Qt Creator should point to a <code>.framework</code> directory.</li>
	<li>Qt Creator will generate code in your project file which will link to the correct framework or library depending on which platform the code is being compiled on.</li>
	<li>Finally, because Apple loves to be different, the glut.h header path is different on OS X.  <strong>You'll need to <code>#include &lt;GLUT/glut.h&gt;</code></strong>, instead of <code>&lt;gl/glut.h&gt;</code> is usually the case.  You can use preprocessor conditional statements to make this inclusion cross-platform:
<pre>#if defined(__APPLE__) || defined(MACOSX)
  #include &lt;GLUT/glut.h&gt;
#else
  #include &lt;gl/glut.h&gt;
#endif</pre>
</li>
</ol>
That's it! If you have any trouble, leave a comment.
