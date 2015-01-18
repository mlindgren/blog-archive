---
layout: post
title: "How to manually send a Pingback"
date: 2015-01-17 21:04:29 -0800
comments: true
categories: [Technology]
---

A couple years ago I stopped using WordPress and
[switched](http://blog.mlindgren.ca/entry/2012/09/23/switching-to-octopress/)
to [Octopress](http://octopress.org/), which is a derivative of
[Jekyll](http://jekyllrb.com/). I continue to be pretty happy with this
decision, but there are some downsides to using a static site generator. One
issue is the lack of support for
[Pingbacks](http://en.wikipedia.org/wiki/Pingback), which are a method supported
by a number of dynamic blogging platforms to notify another blog when you link
to its content. Pingbacks provide a good way to respond to blog posts using
your own blog rather than via commenting; thus, they facilitate "conversation"
between blogs. Unfortunately, if you're using Octopress or Jekyll or another
framework to generate static HTML pages, you won't natively have the ability to
send or receive Pingbacks.

Luckily, Pingbacks can be sent manually with relative ease. Receiving Pingbacks
still won't be possible, though; that's a much harder problem to solve, for a
variety of reasons which aren't relevant to this post.

Before I proceed, I want to note that full credit for this information goes to
[Aaron Parecki](http://aaronparecki.com/); he explains the technique in
[this GitHub Gist](https://gist.github.com/aaronpk/5744879). The reason I'm
reproducing it here is because it was surprisingly hard to find Aaron's Gist,
or anything else of relevance, when I was searching for information on how to
send a Pingback. Hopefully, by posting about it and adding some extra context, I
can make the information more searchable.

First, you'll need to create a copy of this XML file on your local machine:

{% codeblock lang:xml pingback.xml %}
<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
<methodName>pingback.ping</methodName>
<params>
 <param>
  <value>
   <string>http://source/url/here</string>
  </value>
 </param>
 <param>
  <value>
   <string>http://target/url/here</string>
  </value>
 </param>
</params>
</methodCall>
{% endcodeblock %}

Replace the two URLs as appropriate - the first one is the source (i.e. your
blog post), and the second is the target (the post you are linking to).

Now, you just need to `POST` the request to the target server. You'll need to
know the target's XML-RPC endpoint URL for this.  Usually, it will be `/xmlrpc`,
or for WordPress blogs, `/xmlrpc.php`. You can find out for sure by inspecting
the site's source for a `<link rel="pingback" href="..."/>` tag.

You can use cURL to post your request. It's built into most Unix-like operating
systems, but Windows users will have to
[download it](http://curl.haxx.se/dlwiz/?type=bin&os=Win32&flav=-&ver=2000%2FXP),
and if you're on Mac OS X you probably need to have Apple's developer tools
installed.  Just run:

    curl -X POST -d @pingback.xml http://example.com/xmlrpc.php

...and that's it! cURL should print out an XML response from this server with a
message indicating that your Pingback was registered. Of course, to be truly
sure it worked, you should check the target URL and see if your Pingback was
added to the page. (Note that some sites disable Pingbacks.)
