---
layout: post
title: "Why Mozilla Persona Is the Right Answer to the Question of Identity"
date: 2012-10-03 20:37
comments: true
categories: [Technology]
external-url: http://labs.newsint.co.uk/blog/2012/10/why-mozilla-persona-is-the-right-answer-to-the-question-of-identity/
published: false
---

David Somers presents an excellent rundown of what
[Mozilla Persona](http://www.mozilla.org/en-US/persona/) is and why it's
currently the best option for identity and authentication for web services. I've
implemented authentication using Persona in the
[photo album app](http://blog.mlindgren.ca/entry/2012/09/14/social-networks-and-content-ownership/)
I'm working on, and I found it remarkably easy to integrate with my app. And, as
David explains, Persona is also much easier for users to understand than
competing platforms like OpenID. If you're a web developer, I'd urge you to
strongly consider adding Persona support to your application as soon as
possible. It can be done in literally a matter of minutes, and if Persona takes
off the benefit to identity management on the Internet could be huge.

Perhaps my favourite thing about Persona is that it gives me the opportunity to
[implement my own identity provider](https://developer.mozilla.org/en-US/docs/Persona/Implementing_a_Persona_IdP),
which means I can have *complete* control over how I authenticate to any
Persona-consuming service. I could have *n*-factor authentication, with no limit
on the complexity or specificity of the factors besides my own imagination. For
example, I could generate SSL client certificates for each of my machines and
hardcode their thumbprints into my identity provider to ensure that it wouldn't
even service requests from machines I haven't explicitly added.  Or if I wanted
to get creative, I could have my identity provider generate a random temporary
key which I would have to retrieve via SSH (but this is less convenient and
probably also less secure than doing SSL client authentication.) The
possibilities are endless.

Perhaps an end to having dozens upon dozens of different accounts and passwords
for various services,
[each with its own uniquely nonsensical policy](http://me.veekun.com/blog/2011/12/04/fuck-passwords/),
is finally in sight.  Everyone needs to implement Persona&mdash;preferably
yesterday.
