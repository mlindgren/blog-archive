---
layout: post
title: "What are lambdas good for, anyway?"
date: 2011-06-29 22:27
comments: true
external-url:
categories: [Coding]
published: true
---
I had a discussion today about what lambda functions are used for, how they differ from regular functions, and why they're important. I thought I'd reproduce it here, as it seems that lambdas are often thought of as confusing and obscure. The truth is that they're very simple, and also quite powerful. While they can reduce readability if used incorrectly, they do serve an important purpose, and those who understand them should have nothing to fear from them. This discussion pertains specifically to lambda functions as they're implemented in Python, but they're very similar in other languages where they're available.<!--more-->

I think the best way to explain lambdas is with a simple example. Say you want a switch statement. Python doesn’t have them, but what you can do is set up a dictionary and get a value out of it, which is sort of like a switch. However, you can't do this...

``` python
my_dict = {
            1 : my_func1(x, y),
            2 : my_func2(x, y, z)
          }
```

...because the functions will be evaluated when the dictionary is initialized, so if they modify any state (or if they just take a while to execute) you’ll get bad behaviour. Functions are first class objects in Python, so you could insert the functions into the dictionary like so…

``` python
my_dict = {
            1 : my_func1,
            2 : my_func2
          }
```

...but that doesn't work either, because you don’t know how many (or which) arguments you need to pass to the function when you pull it out of the dictionary. So, what to do?

``` python
my_dict = {
            1 : lambda x, y, z: my_func1(x, y),
            2 : lambda x, y, z: my_func2(x, y, z)
          }
```

Lambdas solve the problem because they're evaluated lazily. By being wrapped in
a lambda, the functions won’t be evaluated until the lambda itself is called.
Furthermore, you can use the lambda to normalize the parameters of each
function, so that you can always call it using `my_dict[my_var](x, y, z)`, even though the first function being called doesn’t actually make use of the z parameter. There is no other way to accomplish this in Python, besides a big long ugly list of ifs. (Obviously you’d normally reserve this technique for situations where you have more than two options.)

You can also use the laziness aspect of lambdas to refer to variables which haven't been declared yet, although I'd recommend against that because it can become quite confusing. Anything within the lambda is evaluated only when the lambda is called. 

And there you have it! Those are the basics of lambdas in Python. Of course, the above is only a trivial example of what can be accomplished with lambdas.  If you'd like to learn more, there's a great write-up available <a title="Lambda functions in Python" href="http://www.secnetix.de/olli/Python/lambda_functions.hawk">here.</a>
