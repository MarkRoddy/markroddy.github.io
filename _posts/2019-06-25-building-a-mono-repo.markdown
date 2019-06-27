---
layout: post
title:  "Building Multiple Projects in a Monorepo"
date:   2017-04-06 17:23:53 -0700
categories: jekyll update
---

<div align="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">How do people set up Jenkins for their monorepos (... without converting everything to bazel/pants/etc; just many projects in one repo, with different build requirements, interdependent)?</p>&mdash; Dmitriy Ryaboy (@squarecog) <a href="https://twitter.com/squarecog/status/1143678611406241792?ref_src=twsrc%5Etfw">June 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

# What is in your repo, and what does it build?

The first question to ask your self is what is the nature of the projects in your monorepo, and what does "build" mean for them. Do you:
* Build the contents of Pull Requests and gate merging them on the build passing?
* Automatically do... "things" happen when changes are merged to master? Or what ever the special branch name you picked for your unnecessarily complicated git workflow.
* Generate artifacts out of band from your release process and later have a user promote them? This can be through images such as an AMI or a Docker, more traditional OS packages like RPM's and Debs (the 90's called btw), or language specific packages (jars, wheels, gems, etc).
* Perform an automated deployment of some set of project(s) without a human in the loop? Hopefully after after some level of automated testing has been performed, but hey this your repo. You yolo you.
* Any of the above, but in response to a dependent project being "built" (for that project's definition of build)?
* Or, God forbid, all of the above?

I pinged Dmitriy on Twitter after his initial post and he confirmed that he needed all of the above. You have my condolences, Dmitriy.


# Ideas Informed by Experience

Here are three approaches I've taken to this problem along with some pros and cons I've observed of each.

#### 1. Bash Your Way Through It

Start with a simple bash script that understands project dependencies, looks at the diff of the change set in question, and infers what needs to be built in what order. It grows and grows, and six months later you realize you've written your own build system. Weeping ensues.

**_Pros_**: Your build is defined in your repo and so in theory should be owned by the whole team. Anyone can add a new project and configure its dependencies (hopefully).

**_Cons_**: Loss of build transparency. Questions like "*how often does project Y get built*" or "*how long does project X take to build on average*" are difficult to answer. They likely require parsing build logs and trying to regex out the time to build project Y. Assuming you thought to print that info to begin with.

You are frequently having to remind that one person who answers these questions with data from your Jenkins monojob that "*we build something different on each run*" so "*those numbers are basically meaningless*", and "Seriously Garry, we just talked about this last week*".

#### 2. Directed A-Sickening Graph
Give each of your projects their own Jenkins job. Use the path regex option to restrict builds from triggering unless said project has been modified. Create dependencies between each of these jobs that models the Directed Acyclic Graph of your project dependencies. Setup the [dependency graph plugin](https://wiki.jenkins.io/display/JENKINS/Dependency+Graph+View+Plugin) after a small change to a `README` file unexpectedly causes every project to be rebuilt and deployed. Then react in horror as it shows you the monstrous dependency graph you're created. At least with a shell script no can actually see the mess.


**_Pro_**: Transparency! For better or worse you'll always know when, where, and why each build has been triggered.

**_Cons_**: You've taken complexity that will exist no matter what and moved it from your repo into Jenkins configuration that no one really understands... except maybe you? Adding a new project results in a slew of Jiras for Jenkins setup that are... wait, why are these assigned me? I thought we agree'd people would self service? Wait, I'm the only one who knows how to set these up? Ug, fine.

Also, I haven't used this style of configuration in some time, and if you want handy dandy Pull Request evaluation, it probably won't work anyway.  Unless you attach every build to your PR's and shortcircuit them to not do anyting if a change isn't required. Which, if you do, congratulations, you've just married this approach with the "build your own build system" approach from above.

#### 3. Words Mean What I Say They Mean
Redefine your idea of a build *is* so that you can run through everything on every change. Build every one of your projects for every Pull Request.  How do you get there?
* Pull long running tests in a separate suite that runs "afterwards". Do you need to run them on every Pull Request? Do they even fail for reasons that aren't flakey? Are they even providing value or are you a test horder?
* Use incremental builds instead of full/clean. Yes incremental builds can let issues slip through, but how often does that actually happen in your repo? And is it worth the time it costs to perform them. You can always follow up your incrementals with scheduled full builds throughout the day to catch issues that an incremental would miss.
* Don't use Scala.
* Configure your test harnesses so that any tests that need more than a single digit number of seconds to complete are marked as failures. Delete any repeat offenders. Anyone who objects due to their importance is responsible for making them faster.
* Look the other way when that one project that doesn't have any dependencies in the monorepo some how ended up in its own repo.
* Accept that there is a natural trade off between the speed of your build (and there for your releases) and the amount of things that can happen to give you confidence in the correctness of a pending change.

**_Pros_**: Admittedly, this is not an option for some, dare say many situations. But if you can, teams I've been on that prioritize build/test discipline like this move blazingly fast. Every minute you add between someone hitting merge and their changing reaching production is that much more encouragement to do one Pull Request instead of two or three. This creates larger Pull Requests which give reviewers that much more excuse to drag their feet on getting them done. This incentivizes authors to create even less (but larger) Pull Requests. So your feedback loop keeps turning, productivity it starts burning.

**_Cons_**: Good luck getting here. This is not just a technical decision, but a values decision that your entire team needs to buy into. It only takes one or two folks who aren't comfortable with deleting long running tests before the entire thing is off the rails.

Also, as I previously mentioned, there are lots of types of software for which this just isn't an option. Though if first thought is that you're in this camp I'd ask yourself why? Is the inherent complexity of the software you need to build, the incidental complexity of past design decisions that you'd change if you could, or that you just don't believe it could work?  
