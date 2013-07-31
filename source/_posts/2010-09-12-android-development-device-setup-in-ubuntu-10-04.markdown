---
layout: post
title: "Android Development Device Setup in Ubuntu 10.04"
date: 2010-09-12 17:02
comments: true
external-url:
categories: [Coding]
published: true
---
I spent part of last night setting up my HTC Desire as a development device on my Ubuntu laptop.  This was a somewhat frustrating process, because a lot of the information I came across seem to be outdated and didn't work for me.  Even the <a title="Android documentation for developing on a device" href="http://developer.android.com/guide/developing/device.html">official documentation</a> refers to a version of Ubuntu which is more than two years old.  I did eventually piece together the correct process for Ubuntu 10.04, so I'm documenting that process here in the hope that it will help out other developers in the future.  This guide assumes that you've already installed and configured the <a href="http://developer.android.com/sdk/index.html">Android SDK.</a><!-- more -->
<ol>
	<li>Connect your phone to your computer and open a terminal window.  Run <code>adb devices</code> to confirm that adb can "see" your phone.  You should see something like this:
<pre><code>$ adb devices
List of devices attached
???????????? no permissions</code></pre>
    </li><li>Type <code>lsusb</code> to get a list of attached USB devices.  Look for an entry with your phone's manufacturer; for instance, since I have an HTC device, I get:
<pre><code>$ lsusb
...
Bus 002 Device 003: ID 0bb4:0c87 High Tech Computer Corp. 
...</code></pre>

Note the ID - in my case, <code>0bb4:0c87</code>.  The part before the colon is the vendor ID.  The part after it is the product ID.  Keep these in mind.
</li><li>
You'll now need to create a udev rules file using your preferred text editor.  <a href="http://dimitar.me/how-to-connect-your-android-phone-to-ubuntu-to-do-developmenttestinginstallations-or-tethering/" title="How to connect your Android phone to Ubuntu">This guide</a> lists a number of different file names per-device, but based on my reading of the udev man page, I don't believe the file name is actually important.  The official documentation recommends <code>51-android.rules</code>, which is what I went with.  You'll need to use <code>sudo</code> to get permission to create the file:
<pre><code>$ sudo vi /etc/udev/rules.d/51-android.rules</code></pre>
</li>
<li>
The contents of the file should be as follows:
<pre><code>SUBSYSTEMS=="usb", ATTRS{idVendor}=="[Vendor ID]", ATTRS{idProduct}=="[Product ID]", MODE="0666"</code></pre>
...where the bracketed bits are replaced with the IDs you got earlier from lsusb.  For my HTC Desire, I entered:
<pre><code>SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bb4", ATTRS{idProduct}=="0c87", MODE="0666"</code></pre>
</li>
<li>
Save the file.  Most of the documentation I've come across says that restarting adb and udev is sufficient to give adb access to your device, but <strong>in my case I had to completely restart my computer before adb properly recognized my phone.</strong>  Nevertheless, it's probably best to adb and udev first:
<pre><code>$ sudo restart udev
$ adb kill-server
$ adb start-server</code></pre>
Run <code>adb devices</code> again to see if your phone is recognized.  If you still see <code>???????????? no permissions</code>, you'll probably need to restart your computer.
</li>
<li>
You should now have full access to your phone from adb and Eclipse.
</li>
</ol>

I hope this proves helpful.  If you have any corrections or if this method didn't work for you, leave a comment.
