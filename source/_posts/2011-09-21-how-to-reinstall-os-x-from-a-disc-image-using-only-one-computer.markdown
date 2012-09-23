---
layout: post
title: "How to reinstall OS X from a disc image using only one computer"
date: 2011-09-21 00:17
comments: true
external-url:
categories: []
published: true
---
My internship recently ended, and as such, I had to move from Calgary back to Edmonton, where I'll be completing the fourth year of my Computing Science degree at the University of Alberta. I hope to be able to write more about my internship later, but at the moment I want to share a handy trick I discovered for reinstalling OS X. The reason I bring up the move is that after I arrived in Edmonton, I found myself wanting to reformat my Mac Mini and re-install Snow Leopard, but my Snow Leopard disc was nowhere to be found. Luckily, as a semi-paranoid computer scientist with an abundance of disk space, I keep images of important discs on my network-attached storage device for just such occasions.

<!--more-->

Unfortunately, however, the Snow Leopard disc image is large enough that it can only be burned to a dual-layer DVD. I don't tend to keep writeable dual-layer DVDs lying around, so I needed a solution which required only the disc image itself. There are a number of methods available for installing OS X over the network, but none of them seemed to work for me—perhaps owing to the fact that I had only one Mac. Luckily, I discovered that you can install OS X from a separate partition on the same disk. Here's how:
<ol>
	<li>First, you'll need to re-partition your disk. Open Disk Utility and select the disk you want to use. Be sure to select the disk itself, rather than one of its partitions, which are indented beneath the disk.</li>
	<li>Click the + button to add a new partition, and drag the new partition's section in the disk layout graph upwards until it's large enough to store your operating system disc image. Apply the changes. It may take some time for the disk to be re-partitioned.</li>
	<li>Now, select the new partition from the list on the left. Go to the Restore tab, and restore your operating system disc image to the new partition.</li>
	<li>Once the restore is complete, restart your computer, holding Option (Alt) at start-up. This should bring up a list of available start-up disks. Choose your new partition.</li>
	<li>You should now be at the installer, and you can re-install to your primary partition. Once the installation is complete, delete your installation disc partition to and resize the primary partition to reclaim the extra space.</li>
</ol>
That's it! In retrospect, it's completely obvious, but I haven't seen these instructions anywhere else on the web. This technique might be particularly useful for owners of Macs without disc drives.
