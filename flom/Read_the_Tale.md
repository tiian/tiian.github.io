It's a long tale...
===

Before or later every Linux (and UNIX) system administrator must implement some form of synchronization between two or more commands/scripts.
Typical synchronization scenarios:

* it's necessary to preserve system resources (too much command of the same type should not be executed at the same time)
* it's necessary to preserve data integrity (a file should not read until the writer completed)
* it's necessary to guarantee process consistency (a command must be executed after the completion of another command)

Sometimes the problem is trivial and synchronization can be obtained simply by command concatenation...
Unfortunately, sometimes a system administrator has to synchronize commands/scripts executed by independent scripts. UNIX exists by a lot of time, uhm, a huge amount of time, and a lot of *solutions* have been implemented.

Take a look to Google results (query string "shell script lock"): https://www.google.com/#q=shell+script+lock

Google told me he knows about 3000000 "solutions" of this problem... bingo!

There's another tale...
---

Unfortunately the solutions can be all described as *shell script hacks*. The first result Google gives me is really interesting: http://stackoverflow.com/questions/185451/quick-and-dirty-way-to-ensure-only-one-instance-of-a-shell-script-is-running-at

Some posts are more complex than a C sample for client/server communication over TCP/IP: http://stackoverflow.com/questions/169964/how-to-prevent-a-script-from-running-simultaneously
http://wiki.grzegorz.wierzowiecki.pl/code:mutex-in-bash

I'm not copying the content because I don't think it would be fair, but some examples show 50+ shell code lines of functions necessary to implement what the system administrator needs: "lock"
I do suggest to experiment all that stuff because I think it's a good way to learn a lot about shell scripting, but some time ago I thought: "there should be an easier way".

This is the question:
---

If
 
    nice command
allows me to run *command* with a specific priority, why can't I use

    flom command
to run *command* with a specific synchronization?

That's all!
---
You are reading about a crazy project that's trying to address the above issue while keeping everything **very easy** to use!
