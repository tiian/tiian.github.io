# Use Case 18: sequences
FLoM supports two different types of sequences:

* basic sequences
* transactional sequences

**Basic sequences** are useful to generate unique increasing numbers: every locker obtains a different value from the lock manager.   
**Transactional sequences** are useful to generate unique increasing numbers even if some locker fails to process its value.

Moreover, FLoM sequence resources are multiple: more than 1 locker can obtain the lock at the same time.

## Name syntax

Sequence resource names use a special syntax:

* a prefix that can be *undescore s underscore* or *underscore S underscore*
* a resource name (characters and digits)
* a suffix that must be *square bracket n square bracket* where *n* is a natural number (1, 2, 3...)

### Syntax regular expression

The regular expression is defined in function *global_res_name_preg_init* of source module [src/flom_rsrc.c](https://github.com/tiian/flom/blob/master/src/flom_rsrc.c):

~~~
_[sS]_([[:alpha:]][[:alpha:][:digit:]]*)\\[([[:digit:]]+)\\]
~~~

### Syntax examples

~~~
_s_foo123[4]
_S_barABC[1]
~~~

## The simplest usage
The simplest usage is related to process synchronization on a single non transactional sequence resource.

Prepare a simple script like the following one (edit file */tmp/sh*):

~~~
tiian@ubuntu1004:~$ cat /tmp/sh
#!/bin/sh
echo $1
sleep 3
tiian@ubuntu1004:~$ chmod 754 /tmp/sh
tiian@ubuntu1004:~$ ls -la /tmp/sh
-rwxr-xr-- 1 tiian tiian 26 2016-04-30 22:50 /tmp/sh
~~~

**Note:** basic sequence resources must have a name with prefix "_s_".

execute some times the command "**flom -i 10000 -r _s_a\[1\] -- /tmp/sh**":

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
1
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
2
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
3
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
4
~~~

every execution of the script */tmp/sh* receives a different value.   
With 2 different terminals you can experiment the lock behavior: only one process at a time obtains the lock and the corresponding sequence number:

### Terminal 1 output:

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
1
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
3
tiian@ubuntu1004:~$
~~~

### Terminal 2 output:

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
2
tiian@ubuntu1004:~$ flom -i 10000 -r _s_a[1] -- /tmp/sh
4
tiian@ubuntu1004:~$
~~~

## Notes
Sequences implemented by FLoM are ephemeral resources because FLoM does not manage the persistence. Two aspects must be taken into consideration:

1. *flom* background process terminates by itself after some inactivity: you must explicitly activate a *flom* daemon if you want a never ending process (use command line option "-d, \-\-daemon-lifespan")
2. FLoM resources are subjected to garbage collection after some inactivity: you must explicitly ask an *idle lifespan* if you want to preserve the state associated to a resource for some time (use command line option "-i, \-\-resource-idle-lifespan").

FLoM sequence resources **are not** intended to replace the sequence objects provided by database management systems.

FLoM sequence resources **are intended** to synchronize some processes and to pass them a unique synchronization identificator that's easy to manage and understand.

## The transactional behavior
If you want that every sequence id will be processed, even if some lockers fail, you can use a FLoM *transactional sequence resource*: in case of failure, the sequence id will be rolled back and the next locker will pick-up it again.

To show this behavior, you have do create 2 simple scripts (*/tmp/sh0*, */tmp/sh1*):

~~~
tiian@ubuntu1004:~$ cat /tmp/sh0
#!/bin/sh
echo $1
exit 0
tiian@ubuntu1004:~$ cat /tmp/sh1
#!/bin/sh
echo $1
exit 1
tiian@ubuntu1004:~$ chmod 754 /tmp/sh?
tiian@ubuntu1004:~$ ls -la /tmp/sh?
-rwxr-xr-- 1 tiian tiian 25 2016-04-30 22:19 /tmp/sh0
-rwxr-xr-- 1 tiian tiian 25 2016-04-30 22:19 /tmp/sh1
tiian@ubuntu1004:~$
~~~

*/tmp/sh0* writes the first command line option and terminates correctly, */tmp/sh1* writes the first command line option and fails.

**Note:** transactional sequence resources must have a name with prefix "_S_".

Using the scripts above described, we can see the behavior of a transactional sequence resource:


~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
1
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
1
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
1
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
1
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
1
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
2
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
3
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
4
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
5
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
5
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh1
5
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
5
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
6
tiian@ubuntu1004:~$ flom -i 10000 -r _S_a[1] -- /tmp/sh0
7
tiian@ubuntu1004:~$
~~~

If the command passed to *flom* fails (process exit code not 0), in case of a transactional resource, the sequence id is rolled back and passed to the next lock request.

## Multiple sequence resources

The last part of a sequence resource name is a number surrounded by square brackets:

*   \_s\_a**\[1\]**
*   \_S\_a**\[1\]**

Sequence resources are a special type of numeric resources and they can be multiple; for instance, this resource:

\_s\_foo\[3\]

is a non transactional sequence that can be locked by up to 3 different lockers at the same time: every locker catches a different sequence id, but they can obtain the lock at the same time. A fourth locker will wait until one of the 3 slots will become free.

Multiple sequence resources can be transactional: it can happen that a locker picks up a sequence id that's smaller than the sequence id passed to a previous locker because it's a value rolled back by some other locker.

Using 2 terminals, you can see that both the commands get the lock immediately:

### Terminal 1 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _s_foo[3] -- /tmp/sh
1
tiian@ubuntu1004:~$
~~~

### Terminal 2 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r _s_foo[3] -- /tmp/sh
2
tiian@ubuntu1004:~$
~~~

## Use cases
**How can sequence resources be used?**

* you need to synchronize a bunch of commands/scripts and you want that every command/script can understand its execution order
* you need a simple technology to retrieve a unique sequence id from a command/script
* you want to execute exactly *N* commands/scripts and restart the failed ones
