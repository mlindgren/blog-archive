---
layout: post
title: "Switching to Octopress"
date: 2012-09-23 23:22
comments: true
categories: [Technology]
published: true
---

## Backstory

I've been using Wordpress on various sites that I've maintained for probably the
better part of a decade now. By most metrics, it's still very good blogging
software; I'm not aware of any self-hostable alternative that can match
Wordpress in terms of the union of ease of use, feature set and flexibility that
it provides. That said, lately I've been feeling that perhaps Wordpress is no
longer the right choice for my specific needs. These are my problems with it:

- It's not very good for sharing source code. There are a multitude of plugins
  available for syntax highlighting; the best of them seems to be
  [Alex Gorbatchev's Syntax Highlighter](http://alexgorbatchev.com/SyntaxHighlighter/), 
  which is used by
  [Wordpress.com](http://wordpress.com) and is also what I was using on my blog.
  But I've had problems with the Wordpress converting some of the characters in
  my code to HTML entities (e.g. `&amp;` for & ) which are then printed
  literally by the plugin.
- I find that more often than not Wordpress' visual editor is
  insufficient for the formatting I want in my posts. That forces me to use
  the raw HTML editor. That would be fine, except that Wordpress has a strange
  sort of off-spec way of storing post HTML. Paragraph and line break tags are
  omitted in the editor and the database, and are added when the page is
  actually rendered to the client. This is okay, I guess, and I *think*
  Wordpress is smart enough to not insert extra paragraph tags if you do wrap
  your own paragraphs with them. Still, it leads to some unpredictability with
  regard to how a post with hand-written HTML will actually be rendered.
- A similar but worse problem occurs when you try to insert extra line breaks.
  Even if you're using the raw HTML editor to insert `<br />` tags, Wordpress
  will just get rid of them unless you insert additional non-breaking spaces
  (`&nbsp;`) so that it thinks the lines are non-empty. But! If you switch back
  to the visual editor for any reason, even your `&nbsp;<br />` lines will be
  wiped out. Even for "average" users I can't think of any situation in which
  this "feature" would be desirable. If I put a bunch of extra newlines into
  something I'm writing, it is because *I want exactly that many newlines.* I
  tend to not have much patience for software that thinks it's smarter than
  I am; that is why I still write most of my code in [Vim](http://www.vim.org/)
  and use hand-written makefiles in an age of
  <acronym title="Integrated Development Environment">IDE</acronym>s.
- Never-ending updates. It seems like every time I log in there's a new update
  to download. Wordpress has updates down to a one-click install process, so
  this wouldn't be much of a problem except for the fact that
  every time I have to update Wordpress warns me that I should back up my
  database first. Why? Is there a significant possibility that the update might
  break everything? It's probably just a
  <acronym title="Cover Our Asses">COA</acronym>, and I back up my database
  daily anyway, but this whole aspect of Wordpress makes me uneasy and I don't
  want to have to deal with it any more. Even though I'm covered for backups,
  recovering from a botched update would undoubtedly be a nightmare.
- [Vulnerabilities.](http://en.wikipedia.org/wiki/WordPress#Vulnerabilities) One
  of the big reasons that Wordpress is updated so frequently is that it has a
  long history of security vulnerabilities, and more are still being discovered
  and patched. This also means that if I don't log in frequently enough to
  download new security patches, I'm at risk of having my site compromised,
  loaded up with malware and blacklisted by Google.
- It's written in PHP. [PHP sucks.](http://me.veekun.com/blog/2012/04/09/php-a-fractal-of-bad-design/)
  (I can't link to that enough.) I don't want to be dogmatic about technology
  choices; good software is good software regardless of what language it's
  written in, and my primary desire is always to use whatever is best for the
  task at hand. But lately I've been pretty vocal, both here and in person with
  friends and colleagues, about how bad PHP is. Continuing to rely on it
  undermines my credibility on the issue and makes me a bit of a hypocrite.

  And while this might seem trivial or pedantic, it's
  actually one of the root causes of almost all of the issues above; the weird
  HTML handling, inappropriate entity conversion, and security issues are all
  due, at least in part, to bad design patterns popularized and perpetuated by
  PHP. Wordpress still uses raw non-parameterized SQL queries; blogging software
  built on a modern web framework with a proper
  <acronym title="Object-Relational Mapping">ORM</acronym>
  would not be plagued by SQL
  injection exploits as Wordpress has been historically and probably continues
  to be.
- Wordpress supports themes so that you can customize the apperance of your
  site, but editing them kind of sucks, because they're all written in PHP
  instead of a proper templating language. Years ago I actually wrote a lot of
  PHP, but I still find Wordpress templates pretty ugly and difficult to
  understand; they seem more verbose than should really be necessary for what
  they accomplish, and have, on average, more `<?php ?>` sections than a
  keyboard has keys. (That analogy sucked. Sorry, it's late. What I'm trying to
  say is that there are a lot of them.)

## Octopress

So recently I got it into my head to switch over to
[Octopress.](http://octopress.org/) Octopress bills itself as "a blogging
framework for hackers," which is an attractive description given my
[predilections.](/categories/coding) It uses
[Jekyll](https://github.com/mojombo/jekyll) to generate static pages from a set
of templates and content written in
[Markdown.](http://daringfireball.net/projects/markdown/)<sup>1</sup> It is,
without a doubt, harder to use than to Wordpress, but it has the following
advantages:

- Rendering static pages once and serving dynamic content via AJAX is much more
  efficient than rendering entire pages dynamically for each visitor. This isn't
  really a concern for me because this is a very low-traffic site, and I have an
  entire quad-core <acronym title="Virtual Private Server">VPS</acronym>
  doing basically nothing besides serving this site. That said, it's nice to
  know that when I start doing other stuff on my VPS, I won't be contending for
  cycles with PHP and MySQL, so the site will always be very responsive. Plus,
  wow does the site load fast now (once you have that huge header image cached,
  anyway. I'll fix that soon.) Also not a huge concern for a blog, but I
  didn't expect the difference to even be noticeable, and it definitely is.
- Static pages significantly decrease the attack surface of the site; there is
  no database, so no SQL injection to worry about, and no scripts are being
  executed, so [PHP vulnerabilities](http://www.kb.cert.org/vuls/id/520827)
  aren't a concern.
- Static pages means I get a nicely organized, easily navigable directory
  structure instead of God-awful rewrite rules.
- The default Octopress theme is very well designed and well-suited to
  customization.  I was able to modify it to make it look almost exactly like my
  old custom Wordpress theme did, while retaining the fully fluid layout of the
  Octopress theme&mdash;meaning that the layout will resize to look beautiful at
  any screen resolution, including on mobile devices.
- Octopress has pretty nice built in Github and Twitter widgets.
- Comments are handled by [Disqus](http://disqus.com/), which provides a nicer
  comment system than Wordpress' built in comments.  I was using Disqus anyway
  via a plugin, but now I don't have to worry about keeping the Disqus comments
  synced with the Wordpress database. Incidentally, Disqus provides a very nice
  system to migrate comments to new URLs by uploading a
  <acronym title="Comma Separate Value">CSV</acronym> file mapping old URLs to
  new URLs. They've really thought of everything, so props to them for providing
  a great service.
- All of the blog content is stored on my local machine, which means I can write
  posts in a disconnected environment using Vim, version control them with Git,
  and then push them to my server when they're finished. I have a nice
  deployment setup where everything is stored in a Git repository on the server
  and a post-commit hook copies the updated content to the web root for the blog
  subdomain. All I have to do is `git push` and the blog is instantly updated.
- Since Wordpress uses hand-written SQL queries, it is tightly coupled with
  MySQL and doesn't support any other database. PHP is similarly tied to Apache;
  while it's possible to run it with other servers, it's not particularly easy.
  By ditching Wordpress, I can also get rid of Apache, PHP and MySQL. This is
  nice since I was planning to use nginx and Postgres for my
  [photo album project](/entry/2012/09/14/social-networks-and-content-ownership/)
  anyway.

Those are, give or take a few minor considerations, my reasons for switching
blogging platforms. I'd be remiss to not mention that Octopress does have some
noteworthy disadvantages even for "power users" to whom the above advantages
might sound attractive.
[This post](http://alblue.bandlem.com/2012/02/disadvantages-of-octopress.html
"Disadvantages of Octopress - AlBlue's Blog")
does a good job cataloguing a few of them. On the balance, though, I feel like
it's a good choice for me, and although the switch took me considerably longer
than I expected (the better part of an weekend where I'd expected an evening or
so of work), I'm pretty happy that I did it.

In an upcoming post, I'll discuss the actual process of reconfiguring my server
and importing all of my content (which is what I had planned to do before this
turned into a 1,600 word diatribe against Wordpress.) For now, if I've inspired
you and you're thinking about switching your own Wordpress site to Octopress,
you can take a look at the
[<acronym title="Extract, Transform, Load">ETL</acronym> I wrote](https://github.com/mlindgren/wp2octopress)
to ease the import process.

<sup>1</sup> <small>Did you know that Markdown was created by John Gruber? Until very
recently, I didn't. And here I thought all he did was play apologist for
Apple.</small>
