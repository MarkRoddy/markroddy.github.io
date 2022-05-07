---
layout: post
title:  "First Impressions of PyScript"
date:   2022-05-06 17:23:53 -0700
categories: jekyll update
---


So this happened...
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">📢 Did you hear the news from PyCon!? We are thrilled to introduce PyScript, a framework that allows users to create rich Python applications IN THE BROWSER using a mix of Python with standard HTML! Head to <a href="https://t.co/n4OoeBD46z">https://t.co/n4OoeBD46z</a> for more information. 🧠 💥</p>&mdash; Anaconda (@anacondainc) <a href="https://twitter.com/anacondainc/status/1520447158603890691?ref_src=twsrc%5Etfw">April 30, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


My days of active coding are a bit behind me, but in a past life I was a long time Python user and occassional writer of Javascript. I would describe my Javascript usage as begrudging as it was never something I did often, and when I did the experience was consistently frustrating. You might even say I resented it, though recognized it for what it is: the defacto client side programming language (more modern languages like Typescript that compile to JS aside). So needless to say I was very excited to check this out.

What follows is a summary of picking this tool up and trying out some of the basic tutorials to get a feel for PyScript and where it may (or may not) be useful.

# What's in the tin
PyScript is a set of tools (Anaconda calls it a "framework", ug) that make it easy to run arbitrary Python on the client side (aka the broswer). It lets you seamlessly interleve arbitrary blocks of Python within HTML, similar to a good old fashioned `<script>` tag for Javascript. Where as a classic hello world for Javascript might look like this:
```
<html>
  <body>
    <script>
      alert("Hello World!");
    </script>
  </body>
</html>
```

In PyScript, you get:
```
<html>
  <body>
    <py-script> print('Hello, World!') </py-script>
  </body>
</html>
```

# What it's not...
For completeness, PyScript is doesn't enable running python in the browser. It uses the [Pyodide](https://pyodide.org/en/stable/) project to provide an in browser VM. What PyScript provides is duct tape to make it stupidly easy to embed and run Python code within HTML. Given this framing, I can see how it can be labelled a framework, but... ug, that word.

# Let's Do the Tutorial
I dove into PyScript using the [Getting Started](https://github.com/pyscript/pyscript/blob/main/GETTING-STARTED.md) doc on the project's Github repo. Most getting documentation seems to nudge you this way so it seems a good a place to get started.

Development setup is a breeze because all you need to install is... nothing? Well you'll need a text editor to create HTML files, and a modern webbrowser (they recommend Chrome) to host the VM. But I'll assume you have this sorted if you're read this far.

Next, we have a series of example HTML files interlevened with Python code. These example include:
* [Hello World](https://github.com/pyscript/pyscript/blob/main/GETTING-STARTED.md#your-first-pyscript-html-file) A classic, (similar to above).
* [A Multline Block](https://github.com/pyscript/pyscript/blob/main/GETTING-STARTED.md#the-py-script-tag): This tripped me up a bit, but more on this later.
* [Page Interaction](https://github.com/pyscript/pyscript/blob/main/GETTING-STARTED.md#writing-into-labeled-elements): Showing that we're not just running in the browser, we can also interact with the page/DOM.
* [Dependency Managerment](https://github.com/pyscript/pyscript/blob/main/GETTING-STARTED.md#packages-and-modules): An example using publically available Python modules as well as your own code files.



## Initial Impressions
This is interesting, and potentially useful on day one for certain scientific computing use cases. Especially since PyScript includes some specific tooling that make it incredibly easy to embed data visualizaitons into the DOM from Python libraries.

However, if you're evaluating PyScript from the standpoint of a general purpose Javascript replacement for the browser, there are a few things that should give you pause. First and foremost is the lack of ecosystem. PyScript reminds me of writing javascript before major frameworks, (of which jquery was my first) and before browsers had debugging tools (or extensions like Firebug). In practice, its you and the language in an HTML file.

There is the ability to pull in python packages, but while this is necessary it is not sufficient. Until there is an ecosystem of client oriented libraries, you'll be doing a lot of non-differentiating heavy lifting. This is of course a problem that will most likely solve itself as people start to codify what's needed in library support, and I suspect will move quickly as there is the reference of Javascript history to guide where Python tools and libraries may want to go.

Less clear to me is how important integrated browser support for debugging may be. You certainly don't get *no* support (the DOM inspector and network panels work the same either way), but how much of a step back that there isn't the deep debugger integation? Can this be solved with libraries within the Python runtime? Does it need to be deeply integrated with the browser, if so how long will it take for something like that to show up in Chrome? My guess is it's definitely something that can be lived with. I learned Javascript wthout a debugger. But if you don't *have* to live with it, should you?

### Open Questions
There are a couple of open questions that I wasn't able to get answers to, which I expect I could with a bit more time. For posterity they are:

#### Package Imports
There are examples on importing 3rd party dependencies and single `.py` files. What if I have a package of pure python files? There is a [support ticket](https://community.anaconda.cloud/t/importerror-attempted-relative-import-with-no-known-parent-package/20608) that implies this is possible if you enumerate every file in the package but it requires wonky imports which makes the package unusable outside of PyScript.

#### Evented Model?
Is there an event loop like client Javascript? If so is it the same as the Javascript event loop or a separate loop? Assuming there is either of these, I'd have a host of questions, like, what considerations are needed to not block the loop? 

Python has had support for [asyncio](https://docs.python.org/3/library/asyncio-task.html) since [v3.5](https://peps.python.org/pep-0492/) but how many libraries actually honor this? How much concern should there be that using a 3rd party library might lock up the browser periodically? Please say none. It'd be great if this wasn't a thing to worry about (seeing some of the examples load, I think it may be something to worry about).

#### Complex DOM Interaction
In theory there is the standard browser API's and the Javascript bindings. I haven't dug into this at all to see what the look and feel is. Maybe that's great? Or maybe there's some room for libraries that can move the needle for all of us.



## Things that Tripped Me Up

# Indentation
Space within a `<py-script>` tag is significant (duh), but the identation of the tag is as well. For example, I expected this to be valid:
```
<html>
  <body>
    <py-script>
      def echo(n):
        return n
      value = echo(1.2)
      print(value)
    </py-script>
  </body>
</html>
```
However, this fails with an error about indentation. It took a bit to figure out that whitespace prior to the tag is included in significance. To illustrate, here is how you would fix the above code snippet:
```
<html>
  <body>
    <py-script>
def echo(n):
  return n
val = echo(1.2)
print(f"The Value: {val:.3f}"
    </py-script>
  </body>
</html>
```
Now that I know this, I'm sure I'll recongize it in the future. Rather annoying to have to though.

# Debugging Dependencies
The second example that included dependencies, when I first ran it, I got this error:
```
ModuleNotFoundError: No module named 'matplotlib'
```
This was pretty weird given it's the *second* example, and the first example also included the `matplotlib` dependency. And the first example worked! Why was this different? Well, this second example showed how to load another `.py` file in addition to third party dependencies. Looking at sample code I saw this:
```
<py-env>
  - numpy
  - matplotlib
  - paths:
  - /data.py
</py-env>
```

It appears the environment specificaiton is yaml (or yaml like?), and some whitespace got mangled when I pasted it into the HTML file. Here's what it should look like:
```
<py-env>
  - numpy
  - matplotlib
  - paths:
    - /data.py
</py-env>
```
Note the additional whitespace on the line after `paths`? This seems a reasonable enough mistake, but the error message about `matplotlib` made it difficult to debug. 



## Petty Complaints
And be warned... these *are* petty.

# Constant Code Reformatting
When I open an `*.html` file in my editor, it treats it like an html file. And when it sees:
```
<html>
  <body>
    <py-script>
val = 1.2345
print(f"The Value: {val:.3f}"
    </py-script>
  </body>
</html>
```
It changes it to:
```
<html>
  <body>
    <py-script>
      val = 1.2345
      print(f"The Value: {val:.3f}"
    </py-script>
  </body>
</html>
```
EVERY TIME I CHANGE ANYTHING IN THE `<py-script>` tag.

So every time I changed a line of code block, I had to make an indent adjustment, because my editor would re-indent said line.

Now, you could say this is on me to configure my editor, which, you know, fair. But I'm going to side with my editor here. The formatting of html has felt pretty stable for, oh, 25 years or so? I think my editor's reaction is pretty reasonable. Does it really make sense to diverge from this format here? Is there value to me as a user?

Lastly, is this complaint petty? Sure. I mean, look what section we're in. But it's also a real PITA.

# Labeled Outputs

One feature of the `<py-script>` is you can specify a binding, so the output of your block will be written to a selected HTML element. Kind of cool, but the implementation feels wonky to me. Example:
```html
<div id="plot"></div>
<py-script output="plot">
import matplotlib.pyplot as plt
import numpy as np

x = np.random.randn(1000)
y = np.random.randn(1000)

fig, ax = plt.subplots()
ax.scatter(x, y)
fig
</py-script>
```
The `output="plot"` causes the output of the block of code to be appended to the HTML element with `id="plot"`. But what *is* the output of this script? Well the documentation says this:

> Notice here we're using <py-script output="plot"> as a shortcut, which takes the expression on the last line of the script and runs pyscript.write('plot', fig).

This reminds me of Ruby's [implicit return](https://franzejr.github.io/best-ruby/idiomatic_ruby/implicit_return.html), which makes me reach for the [The Zen of Python](https://peps.python.org/pep-0020/):
> Explicit is better than implicit.

This is solidly a philosophical opinion here so I won't push too hard, but it does feel a bit like spooky action at a distance. Given that, I'd expect that there's a non-trivial savings for that risk of future debuggability issues. Let's review what this saves us again?
> (this) takes the expression on the last line of the script and runs pyscript.write('plot', fig).

So this saves me from typing `pyscript.write('plot', fig)`? A total of 27 characters? But wait, you do have to type the `output` tag and the implicit return value, that's 17 characters. So in the end this misdirection saves us... 10 characters?

I'd have a lot less gripes if there was an explicit return. Alternatively, if there were more things that could be done with data binding, maybe some ways to do more powerful binding? That might make the spooky action feel worth it.

# I had to setup a webserver
I do all my adhoc development on an EC2 instance so I don't have to worry about mucking with local environments, especially since I have multiple computers I use on a regular basis. This gives me a single point of usage. So... I setup an nginx server inside of a Docker container to server all the HTML files. This is a thing that I did to myself, but it did suck the fun out of the first 30 mins of this exercise.


## Analysis and Predictions

# Who this is (and isn't) for
In it's current state, PyScript is clearly a tool for scientific computing use cases. This is unsurprising given it's coming from [Anaconda](https://www.anaconda.com/), a vendor of Python flavored data science software. I'm sure this is will berth some really interesting projects and tools, especially on the visualization side. One of the more interesting things I've read about was a presentation that used the Javascript/Python interoptablility to utilize D3 directly from Python code:
<blockquote class="twitter-tweet" data-conversation="none"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/PyConUS2022?src=hash&amp;ref_src=twsrc%5Etfw">#PyConUS2022</a> <a href="https://twitter.com/pwang?ref_src=twsrc%5Etfw">@pwang</a> Keynote<br>We can now `import d3`<br><br>The pyscript wrapper for d3 was done by a team member in two days ✌️‼️ <a href="https://t.co/DmsNk8zcBl">pic.twitter.com/DmsNk8zcBl</a></p>&mdash; Mariatta 🤦 (@mariatta) <a href="https://twitter.com/mariatta/status/1520436349785976832?ref_src=twsrc%5Etfw">April 30, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Personally, I'm also very optimistic about the value of a Python interpreter in the browser. I've spent more time debugging conda and virtual environment's on other people's laptop than I care to admit. A future where python code can be handed to someone to run, and all the need is a browser is a future I'm excited about.

As for who this isn't for... general purpose python application developers that don't want to learn/use Javascript. I'm guessing the time horizon to "you can write fully featured client applications with Python, as quickly and easily as Javascript" is going to be measured in years. Till then, unless you have a specific use case that this tooling is necessary to solve, you're almost definitely better off getting up to speed on the Javascript ecosystem. Which brings me to my next take away...

# This is not a Javascript killer... yet.
If you have specific technical computing requirements for client side? Go for it. Some other requirement that's massively easy in Python but would require tons of upfront investment to work on Javascript? Do it.

But if you're looking for general purpose client side scripting, for the time being, you're probably better of sticking with what works... for now. Libraries and tools will (likely) start to show up without too much time. Getting browser level support for things like debuggability will probably take much longer. And somewhere between this two points will be a "good enough" intercept that's worth changing this stance. Note that intercept will be specific to you and your needs so don't wait around for someone to tell you "ok, now". 

One potential risk I do see though is the focus on technical computing. Building a full fledged Javscript killer is going to require an ecosystem of tools/libraries/etc that are client side specific. Are these coming? If so, at the same rate as support and tools for technical computing? If not there's a risk PyScript is perceived as more targetted at data scientists and other technical computing use cases. If that perception emerges, warranted or not, it could prove a hinderance to general purpose aadoption, which would exaserbate the ecosystem issues previously mentioned. Time will tell how founded this concern is.


