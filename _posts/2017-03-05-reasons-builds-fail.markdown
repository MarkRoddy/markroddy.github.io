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
* The database we didn’t know our tests hit went down [[0]](#footnote-0).
* We switched JavaScript tool chains, but the tool chain isn’t installed on the build box.  
* Our build instance died. We reprovisioned it, but our JavaScript toolchain is missing again. [[1]](#footnote-1)
* Github SSH keys were rolled. They weren’t rolled on the build box.
* We hit the file system’s maximum number of subdirectories in the `/tmp` directory [[2]](#footnote-2). Bonus: This can’t be fixed by running `rm -rf /tmp/*`. [[3]](#footnote-3)
* The build process hit the max number files of the sysctrl kernel limit. [[4]](#footnote-4)
* The partition where the builds are executed is full. Bonus: There is only one partition, now the machine is unreachable. 
* Github is down again.
* Someone ran `sudo rm -rf /var/lib/jenkins`. I've personally done this. Twice. [[5]](#footnote-5)
* Double-digit GB files were committed to the code base. [[6]](#footnote-6)
* The change works on OS X, but the build runs on Linux. [[7]](#footnote-7)
* The agent on the Windows build machine is down. Bonus: no one knows the password for the Windows build box.
* There are symlinks in your git repo.
* The commit includes a new configuration parameter, with defaults for the dev and prod environments, but not for test.
* An intern encountered their first merge conflict. [[8]](#footnote-8)
* The build machine has a different JVM version than the development environment. [[9]](#footnote-9)
* Our version of Ruby was updated when someone installed security updates. [[10]](#footnote-10)
* Github is down.... but just the ssh endpoints.  [[11]](#footnote-11)
* Time out. The build hadn’t written to`stdout` in a while, because it was stuck.
* Time out. The build hadn’t written to `stdout` in a while, because `stdout` was being buffered. [[12]](#footnote-12)
* The IAM permissions clean up project. [[13]](#footnote-13)
* TravisCI is down.
* TravisCI changed the default version of XXXXX. [[14]](#footnote-14)
* That thing, where you’re changing your .travis.yml, and it takes you 5-10 commits to get a passing build.
* Left-Pad
* You build PRs and commits to master, but they have different config files. [[15]](#footnote-15)
* The build was already broken. [[16]](#footnote-16)
* It required being run with `sudo` to pass. [[17]](#footnote-17).
* Github is up!  But only the ssh endpoints.  The website is down, and we use their OAuth provider to log into Jenkins. [[18]](#footnote-18)
* A unit test found bug in our software. [[19]](#footnote-19)

This list should neither be considered comprehensive nor representative of anyone’s experience but my own. Building software is hard, and all any of us can do is try our best. [[20]](#footnote-20) 


## Footnotes:
0. <a name="footnote-0"></a> Three months ago someone added a test which required the redis instance in the “demo” environment the product team uses to show off new features to the execs. No one new about the dependency until the execs declared that the demos are a waste of time, and the ops team tore down the demo environment.
1. <a name="footnote-1"></a> We installed it manually to get the build working.
2. <a name="footnote-2"></a>Somewhere in core hadoop, possibly HDFS, a temp directory is created every time a class is instantiated. That class was instantiated multiple times for every test case. None were cleaned up because no one knew it was happening. Never had the time to track down the source. Added a cron to delete anything older than a week.
3. <a name="footnote-3"></a> Bash will attempt to expand the ‘*’ wildcard to include every file and directory present, which is more than its buffer has room for and tell you “Argument list too long”.  There are some workarounds though.
4. <a name="footnote-4"></a> Or open files, or processes, etc. 
5. <a name="footnote-5"></a> Moving the Jenkins home directory to a separate volume with more space because the main partition was filled up. Moved the home dir to `/var/lib/jenkins-backup`. Mounted the volume at `/var/lib/jenkins`, and ran `cp /var/lib/jenkins-backup/* /var/lib/jenkins`. A few weeks later I went to delete the backup, but fat fingered the command and deleted the contents of the volume. Only lost a few weeks of history thankfully.
6. <a name="footnote-6"></a> Multiple stellar outcomes here. A full disk? Timeouts on the changes being pulled down?
7. <a name="footnote-7"></a> Go allows for platform specific code to be defined in separate files using a naming convention to identify the platform. There’s no requirement you provide a version of the file for platforms you aren’t running. Developer committed only an OSX file.
8. <a name="footnote-8"></a> They commited the merge conflict TODO: link to text of commited merge conflict
9. <a name="footnote-9"></a> There’s an upgrade in progress with a staged roll out to development, then staging, and then test. The build system wasn’t accounted for.
10. <a name="footnote-10"></a> Because we installed it via the linux distro’s package manager.
11. <a name="footnote-11"></a> So... we can see our changes, we just can’t build them.
12. <a name="footnote-12"></a> Python
13. <a name="footnote-13"></a> The AWS keys installed on the build box lost the ability to upload the build artifacts to S3. Could still perform LIST operations though, so we had that going for us.
14. <a name="footnote-14"></a> We didn’t specify a version, even though we could have.
15. <a name="footnote-15"></a> The PR build worked. The build off of master, sadly, did not.
16. <a name="footnote-16"></a> Since Friday last week. It’s Wednesday.
17. <a name="footnote-17"></a> We can’t tell why this build failed.
18. <a name="footnote-18"></a> No one really seems to know why.
19. <a name="footnote-19"></a> It’s been known to happen on occasion.
20. <a name="footnote-20"></a> Some of the situations described have been fictionalized to protect the innocent as well as others that did not try their best.
