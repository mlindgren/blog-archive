---
layout: post
title: "Photo App Progress Update"
date: 2012-11-14 22:58
comments: true
categories: [Coding]
---

In September I [wrote about](/entry/2012/09/14/social-networks-and-content-ownership/)
my intention to develop a web application to share my photos. I made good
progress on it throughout September and early October, but for various reasons I
haven't been able to work on it much in the past month or so. Last weekend,
though, I was able to solve some blocking issues which were preventing me from
doing an alpha release. Those being dealt with, I've now got a few of my albums
up at [photos.mlindgren.ca](http://photos.mlindgren.ca/); take a look and leave
a comment if you have any feedback.

I haven't released the source code yet as there's still a great deal of work to
be done, and I'm generally of the belief that dumping a bunch of unfinished,
messy code on Github with the hope that the community will sort it out is of
little benefit to anyone. My goal at this point is to do a beta release in a
two to three months or so and release the code at that point. That timeline is
anything but firm, though. This project is turning out to be much more work than
I expected (as projects are wont to do), and I expect to be fairly busy over the
next couple months.

Anyway, with the alpha version up and running, this seems like a good
opportunity to reflect on some of the issues I've faced that complicated the
project, and on what remains to be done. <!-- more -->

## Wand and the OOM Killer

I mentioned in my
[post on Python imaging libraries](/entry/2012/09/14/the-state-of-python-imaging/)
that I'm using [Wand](http://dahlia.kr/wand/index.html) to resize and transform
uploaded photos. It's very convenient and easy to use, and from a
maintainability and ease of use perspective I much prefer using Python bindings
for ImageMagick to shelling out to the ImageMagick executable. One
of my goals for this project is that other people should be able to install it
on their own servers with relative ease, and I suspect relying on ImageMagick
executables for image processing would lead to cross-platform and configuration
issues.

But, Wand is not yet a mature library and it has some significant issues of its
own. First and foremost, when I started using it it had some major memory
leaks. I found and fixed more than 10 leaks and submitted a pull request which
was merged into the main repository&mdash;however, there hasn't yet been an
official release including my fixes yet, so pip installs and downloads from the
Wand website will still include those leaks.

I'm not sure if I got all of the leaks or if more still remain. The ones I fixed
were discovered mostly through ad-hoc testing and reading the code. I've been
running my app for several days now and the memory consumption when it's idle
has stayed flat, so I believe I've removed at least the most common leaks, but I
haven't had a chance to sit down with Valgrind yet and run the Wand unit tests
through it to make sure that I got *all* of the leaks. I'll have to do that
before I do any sort of public release of my software;
I don't want to rely on a leaky library.

Beyond the leaks, I ran into another memory issue which proved problematic:
resizing photographs can use a great deal of memory, which is extremely limited
on a reasonably-priced <acronym title="Virtual Private Server">VPS</acronym>.
I have limited knowledge about image processing in general and ImageMagick in
particular, but I'm not aware of any resizing algorithms which can operate
directly on compressed image data, and if any such algorithms exist, ImageMagick
doesn't use them. That being the case, to resize an image, the entire image has
to be loaded into memory decompressed. For an 8 megapixel photograph at 24 bits
per pixel, that means allocating approximately 183MB of memory.

On a 512MB VPS, this means that trying to resize three images
concurrently&mdash;or even sequentially in some cases, since Python isn't
guaranteed to immediately relinquish allocated memory to the operating system
when objects resident in that memory are "deleted"&mdash;practically guarantees
that the process will be killed by the
<acronym title="Out Of Memory">OOM</acronym> killer.

This goes hand in hand with another complication, which is that resizing images
is too slow to be done in the same server thread that's handling requests and
generating responses. For each image that's uploaded, I save several different
sizes for different purposes, and due to the aforementioned memory constraints I
can't even resize them concurrently. Resizing each image several times in the
responding thread drastically reduces throughput for the client
since it incurs a significant wait time for the server to process each image
after it's uploaded.

## Enter Subprocess

So here's the solution I've come up with for the time being, and I'm very unsure
whether or not it's actually a good solution, so please do let me know in the
comments if there's a better way to do this.  When a new image is uploaded, the
responding thread saves the image in the appropriate directory and adds it to
the database, but does not resize it.  For each of the sizes that the image
needs to be resized to, it adds a task to a
[synchronous queue](http://docs.python.org/2/library/queue.html). The queue is
consumed by a different thread which runs throughout the lifetime of the
application. That thread maintains a (thread-local) queue of
[subprocesses](http://docs.python.org/2/library/multiprocessing.html) with a
configurable maximum length. When a new resizing task is consumed, if there is
room in the subprocess queue, a new subprocess is spawned to resize the image.
Otherwise, the thread joins the subprocess at the front of the queue so as to
block until there's room to spawn a new subprocess. The exit code of completed
processes is checked to ensure that the resize was successful; in case of
failure, the task is re-added to the back of the task queue.

This guarantees that the application server itself will never be killed by the
OOM killer since it uses minimal memory. It never has to load images into
memory; all of that is done in the resizing subprocesses.  By configuring the
maximum number of concurrent subprocesses, one can scale this solution according
to available resources: if you have plenty of RAM, you can resize many images
concurrently and get through the queue faster, or if you have very little, you
can limit the queue to one or two processes to minimize the chance that anything
will be OOM killed.

The subprocess spawning notwithstanding, this is a pretty standard task queue
model, so you might be wondering why I didn't use something like
[Celery](http://celeryproject.org/) in conjunction with a real message queue.
It comes down to minimizing the number of external dependencies and maximizing
ease of use. My project already depends on a number of libraries which will have
to be installed by users. To the greatest extent that I can do so without
compromising on features, I want to avoid complicating the installation further
by adding more dependencies. Celery is particularly difficult to install as it
requires installing and configuring a broker and managing processes for both the
broker and Celery itself. Were I writing a large-scale service and maintaining a
single backend for hundreds of users, it would probably be the right choice, but
as I am writing software for users to install on their own servers, it is
probably not.

This method works fairly well and it's the best I've come up with so far, but
something about it makes it feel more like a hack than a well thought out and
robust solution. I also wonder if it defeats the purpose of using Wand in the
first place; I use it for very little other than resizing, and since I'm going
to the trouble of spawning subprocesses to do that, perhaps it would actually be
better to just directly invoke the ImageMagick binaries. As mentioned above, I
have concerns about what impact that would have in different environments, but I
haven't really validated those concerns; they're just hunches.

## The Road Ahead

So I've got a workable, if not perfect, solution to one of the biggest problems
I encountered... but as I mentioned previously, there's still much work to be
done. Here are a few (but not all) of the things I'd like to do before I release
anything publicly:

- **Lots of UI work.** Right now the app works very poorly on phones and iPads.
  I also need to figure out how to make certain features more discoverable, and
  although I'm happy with how albums look right now, there are still some extra
  touches that I want to add which I haven't yet been able to (because CSS and
  HTML suck.)
- **Better administrative tools.** Uploading works great, but I have very
  barebones interfaces for editing albums right now. There is currently no means
  of rotating a photo from the editing interface (although doing so should
  almost never be necessary since the app automatically rotates images which
  have EXIF orientation data.)
- **Mirroring and proxying to cloud storage services** such as Azure and S3. VPS
  resources are very expensive; extra disk space on some popular hosts runs
  around a dollar per gigabyte per month. That's ten times the cost of disk
  space on the cloud storage services I've looked at. Cloud-hosted images might
  also improve load speeds for visitors.
- **Comments.** This one is pretty big, obviously, but I haven't added it yet
  because I can't decide on how best to handle it. I'd like to avoid writing my
  own comment system and requiring potential commenters to sign up for yet
  another account. (I'd use Persona for commenter identification and
  authentication just as I do for the admin login, but very few people currently
  have Persona accounts or even know what it is.) I like Disqus, but I'm not
  sure it would integrate well with the minimalist interface I've designed.
- **Geotagging and maps.** I'm already reading GPS data from EXIF tags where
  available, but now I need to do something with it. I'd like to add the ability
  to manually add locations to photos and albums, and to have a map that shows
  where photos were taken.

Those five items represent maybe a quarter to a third of the work I still have
planned, so to repeat myself again, there's a lot of work to do. But I'm very
happy with the progress I've made so far, and I feel confident that I can get
this finished. It's just a matter of when.
