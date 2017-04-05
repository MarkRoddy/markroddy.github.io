---
layout: post
title:  "Reasons Software Builds Fail"
date:   2017-03-05 17:23:53 -0700
categories: jekyll update
---

A research lab studying software engineering reached out to me recently to take a survey on Continuous Integration tools.  One of the questions they asked was if I could name a few ways I’ve seen software builds fail.  

They got more than a few.

* Github is down.
* The database our tests are supposed to hit went down.
* The database we didn’t know our tests hit went down [[1]](#footnote-1).
* We switched JavaScript tool chains, but the tool chain isn’t installed on the build box.  
* Our build instance died. We reprovisioned it, but our JavaScript toolchain is missing again. [[2]](#footnote-2)
* Github SSH keys were rolled. They weren’t rolled on the build box.
* We hit the file system’s [maximum number of subdirectories](https://ext4.wiki.kernel.org/index.php/Ext4_Howto#Sub_directory_scalability) in the `/tmp` directory [[3]](#footnote-3). Bonus: This can’t be fixed by running `rm -rf /tmp/*`. [[4]](#footnote-4)
* The build process hit the max number files of the sysctrl kernel limit. [[5]](#footnote-5)
* The partition where the builds are executed is full. Bonus: There is only one partition, now the machine is unreachable. 
* Github is down again.
* Someone ran `sudo rm -rf /var/lib/jenkins`. I've personally done this. Twice. [[6]](#footnote-6)
* Double-digit GB files were committed to the code base. [[7]](#footnote-7)
* The change works on OS X, but the build runs on Linux. [[8]](#footnote-8)
* The agent on the Windows build machine is down. Bonus: no one knows the password for the Windows build box.
* [There are symlinks in your git repo](https://issues.jenkins-ci.org/browse/JENKINS-22376).
* The commit includes a new configuration parameter, with defaults for the dev and prod environments, but not for test.
* An intern encountered their first merge conflict. [[9]](#footnote-9)
* The build machine has a different JVM version than the development environment. [[10]](#footnote-10)
* Our version of Ruby was updated when someone installed security updates. [[11]](#footnote-11)
* Github is down.... but just the ssh endpoints.  [[12]](#footnote-12)
* Time out. The build hadn’t written to`stdout` in a while, because it was stuck.
* Time out. The build hadn’t written to `stdout` in a while, because `stdout` was being buffered. [[12]](#footnote-12)
* The IAM permissions clean up project. [[14]](#footnote-14)
* TravisCI is down.
* TravisCI changed the default version of XXXXX. [[15]](#footnote-15)
* That thing, where you’re changing your .travis.yml, and it takes you 5-10 commits to get a passing build.
* [Left-Pad](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/).
* You build PRs and commits to master, but they have different config files. [[16]](#footnote-16)
* The build was already broken. [[17]](#footnote-17)
* It required being run with `sudo` to pass. [[18]](#footnote-18).
* Github is up!  But only the ssh endpoints.  The website is down, and we use their OAuth provider to log into Jenkins. [[19]](#footnote-19).
* A unit test found bug in our software. [[20]](#footnote-20)

This list should neither be considered comprehensive nor representative of anyone’s experience but my own. Building software is hard, and all any of us can do is try our best. [[21]](#footnote-21) 


## Footnotes:
0. <a name="footnote-1"></a> Three months ago someone added a test which required the redis instance in the “demo” environment the product team uses to show off new features to the execs. No one new about the dependency until the execs declared that the demos are a waste of time, and the ops team tore down the demo environment.
0. <a name="footnote-2"></a> We installed the build tool chain  manually to get the build working.
0. <a name="footnote-3"></a>Somewhere in core hadoop, I think HDFS, a temp directory is created every time a class is instantiated, but it was never cleaned up. That class was instantiated multiple times for every test case, and the were never cleaned up because no one realized it was happening. Never had the time to track down the source. Added a cron to delete anything older than a week.
0. <a name="footnote-4"></a> Bash will attempt to expand the ‘*’ wildcard to include every file and directory present, which is more than its buffer has room for and tell you “Argument list too long”.  [There are some workarounds though](http://www.stevekamerman.com/2008/03/deleting-tons-of-files-in-linux-argument-list-too-long/).
0. <a name="footnote-5"></a> Or open files, or processes, etc. 
0. <a name="footnote-6"></a> Moving the Jenkins home directory to a separate volume with more space because the main partition was filled up. Moved the home dir to `/var/lib/jenkins-backup`. Mounted the volume at `/var/lib/jenkins`, and ran `cp /var/lib/jenkins-backup/* /var/lib/jenkins`. A few weeks later I went to delete the backup, but fat fingered the command and deleted the contents of the volume. Only lost a few weeks of history thankfully.
0. <a name="footnote-7"></a> Multiple stellar outcomes here. A full disk? Timeouts on the changes being pulled down?
0. <a name="footnote-8"></a> Go allows for platform specific code to be defined in separate files using a naming convention to identify the platform. There’s no requirement you provide a version of the file for platforms you aren’t running. Developer committed only an OSX file.
0. <a name="footnote-9"></a> They commited the merge conflict TODO: link to text of commited merge conflict
0. <a name="footnote-10"></a> There’s an upgrade in progress with a staged roll out to development, then staging, and then test. The build system wasn’t accounted for.
0. <a name="footnote-11"></a> Because we installed it via the linux distro’s package manager.
0. <a name="footnote-12"></a> So... we can see our changes, we just can’t build them.
0. <a name="footnote-13"></a> [Python](https://www.google.com/search?q=python+buffered+stdout).
0. <a name="footnote-14"></a> The AWS keys installed on the build box lost the ability to upload the build artifacts to S3. Could still perform LIST operations though, so we had that going for us.
0. <a name="footnote-15"></a> We didn’t specify a version, even though we could have.
0. <a name="footnote-16"></a> The PR build worked. The build off of master, sadly, did not.
0. <a name="footnote-17"></a> Since Friday last week. It’s Wednesday.
0. <a name="footnote-18"></a> We can’t tell why this build failed.
0. <a name="footnote-19"></a> No one really seems to know why.
0. <a name="footnote-20"></a> It’s been known to happen on occasion.
0. <a name="footnote-21"></a> Some of the situations described have been fictionalized to protect the innocent as well as others that did not try their best.
