It's a long tale...
===

Before or later every Linux (and UNIX) system administrator must implement some form of synchronization between two or more commands/scripts.
Typical synchronization scenarios:

* it's necessary to preserve system resources (too much command of the same type should not be executed at the same time)
* it's necessary to preserve data integrity (a file should not read until the writer completed)
* it's necessary to guarantee process consistency (a command must be executed after the completion of another command)

Sometimes the problem is trivial and synchronization can be obtained simply by command concatenation...
Unfortunately, sometimes a system administrator has to synchronize commands/scripts executed by independent scripts. UNIX exists by a lot of time, uhm, a huge amount of time, and a lot of *solutions* have been implemented.

Take a look to Google results: query string [shell script lock](https://www.google.com/#q=shell+script+lock)

Google told me he knows about 15800000 "solutions" of this problem... bingo!

But are them really **solutions**?! Unfortunately not: most of the time, they are great hacks that solve a specific problem, not a *class of problems".

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
