---
layout: post
title: "Android NDK Stack Trace Script"
date: 2011-04-09 16:30
comments: true
external-url:
categories: [Coding]
published: true
---
I've finally started doing some Android development in earnest, and because I have a stubborn dislike of Java, I've decided to use the <a href="http://developer.android.com/sdk/ndk/index.html">Native Development Kit</a>.  The native development kit allows you to write "performance-critical" parts of your application in C or C++ code, which is then compiled to platform-native ARM or MIPS instructions.  The NDK is something of an unfriendly beast; it makes interaction with most regular Android libraries very difficult, and debugging native code on Android is none too easy.  Although Google provides a script which can attach gdb to your native code, I've not yet figured out how to use it properly; every time I execute it, gdb complains about missing debugging symbols and libraries and seems to be unable to do anything useful.  (I don't imagine it's too difficult, but I haven't spent much time on it—I've not needed to do much debugging yet.)<!--more-->

When your native code crashes due to a segmentation fault or some other similar problem, though, you do get a nice stack trace in the Android debugging log, which you can view with <code>adb logcat</code>.  Unfortunately, the stack trace only gives you memory addresses, and you need to use another script to convert those memory addresses to lines of code.  That script is already included with the NDK, but it's a bit of a pain to use.  To that end, I've put together a simple shell script which glues everything together. I should mention that this is designed to be used with Linux or Mac OS X; if you'd like to do something similar on Windows, I'm sure it won't be hard, but you're on your own.  Here's how to use it:
<ol>
	<li>Create a folder in your project called "debug" (or whatever name you'd like)</li>
	<li>Create a file called <code>debug.sh</code>, consisting of the following:

``` bash
#!/usr/bin/env bash
NDK_ROOT/toolchains/arm-eabi-4.4.0/prebuilt/linux-x86/bin/arm-eabi-objdump -S PATH_TO_SOLIB > SOLIB_NAME.arm
adb logcat -d > logcat.txt
python parse_stack.py SOLIB_NAME.arm logcat.txt
```

	<li><code>chmod +x debug.sh</code> so that it's executable</li>
        <li>Run the script with <code>./debug.sh</code> each time you need to see a stack trace</li>
</ol>
Obviously there are a few things you'll need to replace in the above code.  <code>NDK_ROOT</code> should be the path to your Android NDK installation.  <code>PATH_TO_SOLIB</code> should be the path to your compiled shared object library.  Usually this will be in your project folder under <code>obj/local/armeabi</code>.  <code>SOLIB_NAME</code> can really be whatever name you want, but I liked to name the <code>.arm</code> file after the corresponding <code>.so</code> file for consistency.

You should repeat line 2 for each shared object library you need to debug in your project, and repeat line 4 for each corresponding <code>.arm</code> file.  However, note that the <code>arm-eabi-objdump</code> process is quite slow, so if you have any large <code>.so</code> files which don't change frequently, you can skip dumping them to improve speed.  Just remember that if they do change, you'll need to dump them again to get accurate results.
