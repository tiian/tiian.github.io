# Use Case 19: (distinct) timestamps

FLoM supports **distinct timestamps** resources.   
They are useful to generate distinct timestamps: every locker obtains a different value from the lock manager.   
In the event of two or more lockers requiring a timestamp inside the same time windows, some lockers will wait some amount of time before obtaining a valid distinct timestamp.

Moreover, FLoM timestamp resources are multiple: the number of concurrent lockers is limited.

## Name syntax

Timestamp resource names use a special syntax:

* a prefix that must be *undescore t underscore*
* a timestamp flom format
* a suffix that must be *square bracket n square bracket* where *n* is a natural number (1, 2, 3...)

The timestamp flom format can contain:

* characters
* digits
* *conversion specification* expressed using the syntax documented in STRFTIME(3)
* *sub second conversion specification* expressed using the following available formats: #f (tenths of second), #ff (hundredths of second), #fff (milliseconds) and so on until #ffffff for microseconds
* 2 special delimiters to obtain a human readable value: "." (dot), ":" (colon)

### Syntax regular expression

The regular expression is defined in function *global_res_name_preg_init* of source module [src/flom_rsrc.c](https://github.com/tiian/flom/blob/master/src/flom_rsrc.c):

~~~
_[t]_([%#[:alpha:]][%#\\.\\:[:alpha:][:digit:]]*)\\[([[:digit:]]+)\\]
~~~

### Syntax examples

~~~
_t_foo.%D.%T[4]
_t_bar:%S#fff[1]
~~~

## The simplest usage
The simplest usage is related to process synchronization on a single timestamp resource.

Prepare a simple script like the following one (edit file */tmp/sh*):

~~~
tiian@ubuntu1004:~$ cat /tmp/sh
#!/bin/sh
echo $1
sleep 3
tiian@ubuntu1004:~$ chmod 754 /tmp/sh
tiian@ubuntu1004:~$ ls -la /tmp/sh
-rwxr-xr-- 1 tiian tiian 26 2016-07-05 10:23 /tmp/sh
~~~

execute some times this command
~~~
flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh :
~~~

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh 
bar.07/05/16.10:39:01
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh 
bar.07/05/16.10:39:05
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh 
bar.07/05/16.10:39:08
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh 
bar.07/05/16.10:39:12
~~~

every execution of the script */tmp/sh* receives a different timestamp value.   

Due to the **sleep 3** command inside */tmp/sh* script, you will not be able to get consecutive timestamps.   
Open a second terminal, and execute the same command in both terminals:

### Terminal 1 output:

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh
bar.07/05/16.10:51:46
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh
bar.07/05/16.10:51:54
tiian@ubuntu1004:~$
~~~

### Terminal 2 output:

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh
bar.07/05/16.10:51:47
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_bar.%x.%X[4]" -- /tmp/sh
bar.07/05/16.10:51:55
tiian@ubuntu1004:~$
~~~

Even if you switch fast from *terminal 1* to *terminal 2*, you will not be able to get the same timestamp.

## Notes
Timestamps implemented by FLoM are ephemeral resources because FLoM does not manage the persistence. Two aspects must be taken into consideration:

1. *flom* background process terminates by itself after some inactivity: you must explicitly activate a *flom* daemon if you want a never ending process (use command line option "-d, \-\-daemon-lifespan")
2. FLoM resources are subjected to garbage collection after some inactivity: you must explicitly ask an *idle lifespan* if you want to preserve the state associated to a resource for some time (use command line option "-i, \-\-resource-idle-lifespan"). For *distinct timestamp* resource **it's strongly suggested** to specify a value larger than the minimum timestamp resolution.

FLoM timestamp resources **are not** intended to replace the timestamp values provided by some relational database management systems.

FLoM timestamp resources **are intended** to synchronize some processes and to pass them a unique timestamp that's easy to manage and understand.

## Minimum time resolution

The timestamp **must** specify a conversion specification that changes at least once per hour: take a look to the below example (the meaning of %d, %H, %M, %S are described in STRFTIME man page):

~~~
tiian@ubuntu1004:~$ flom -r "_t_error.%d:%H:%M:%S[1]" -- echo
error.05:12:04:28
tiian@ubuntu1004:~$ flom -r "_t_error.%d:%H:%M[1]" -- echo
error.05:12:04
tiian@ubuntu1004:~$ flom -r "_t_error.%d:%H[1]" -- echo
error.05:12
tiian@ubuntu1004:~$ flom -r "_t_error.%d[1]" -- echo
flom_client_lock: ret_cod=-27 (ERROR: the server has unilaterally closed the connection)
tiian@ubuntu1004:~$
~~~

## Multiple timestamp resources

The last part of a timestamp resource name is a number surrounded by square brackets:

~~~
_t_bar.%D.%T[4]
_t_foo:%S#fff[3]
~~~

Timestamp resources are a special type of numeric resources and they can be multiple; for instance, this resource:

~~~
_t_foo:%S#fff[3]
~~~

is a timestamp that can be locked by up to 3 different lockers at the same time: every locker catches a different timestamp, but they can obtain the lock at the same time. A fourth locker will wait until one of the 3 slots will become free.

Create a new simple script with a bigger delay, like the following one (edit file */tmp/sh1*):

~~~
tiian@ubuntu1004:~$ cat /tmp/sh1
#!/bin/sh
echo $1
sleep 25
tiian@ubuntu1004:~$ chmod 754 /tmp/sh1
tiian@ubuntu1004:~$ ls -la /tmp/sh1
-rwxr-xr-- 1 tiian tiian 27 2016-07-05 11:06 /tmp/sh1
~~~

Using 4 terminals, you can see that 3 commands can get the lock immediately, but the fourth one has to wait:

### Terminal 1 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_foo:%S#fff[3]" -- /tmp/sh1
foo:47.865
tiian@ubuntu1004:~$
~~~

### Terminal 2 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_foo:%S#fff[3]" -- /tmp/sh1
foo:49.547
tiian@ubuntu1004:~$
~~~

### Terminal 3 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_foo:%S#fff[3]" -- /tmp/sh1
foo:51.187
tiian@ubuntu1004:~$
~~~

### Terminal 4 output

~~~
tiian@ubuntu1004:~$ flom -i 10000 -r "_t_foo:%S#fff[3]" -- /tmp/sh1
foo:12.867
tiian@ubuntu1004:~$
~~~

**Explanation of the above behavior:**

* *terminal 1*: the first locker gets timestamp "foo:47.865" (47 seconds, 865 milliseconds) and keeps a 25s lock
*  *terminal 2*: the second locker gets timestamp "foo:49.547" (49 seconds, 547 milliseconds) and keeps a 25s lock
*  *terminal 3*: the third locker gets timestamp "foo:51.187" (51 seconds, 187 milliseconds) and keeps a 25s lock; this is the last available lock because the resource specifies *\[3\]*
*  *terminal 4*: the fourth locker **waits for an available lock**, gets timestamp "foo:12.867" (12 seconds, 867 milliseconds) and keeps a 25s lock.

If you compare the first timestamp ("foo:47.865") with the fourth timestamp ("foo:12.867"), you can see they differ by 25 seconds and 2 milliseconds:

* 25 seconds are related to first locker *activity* ("sleep 25")
* 2 milliseconds are the consequence of the synchronization overhead

## Concurrency

To create some real concurrency, we need a different script with an infinite loop.

Create a new script, like the following one (edit file */tmp/sh2*):

~~~
tiian@ubuntu1004:~$ cat /tmp/sh2
#!/bin/sh
while (true)
do
	flom -i 10000 -r "_t_time.%D.%T[10]" -- echo
done
tiian@ubuntu1004:~$ chmod 754 /tmp/sh2
tiian@ubuntu1004:~$ ls -la /tmp/sh2 
-rwxr-xr-- 1 tiian tiian 76 2016-07-05 11:25 /tmp/sh2
~~~

Just as an example, execute the script in 4 terminals and stop all the instances using \[CTRL\]+\[C\]; here's the result:

### Terminal 1 output

~~~
tiian@ubuntu1004:~$ /tmp/sh2 
time.07/05/16.11:30:08
time.07/05/16.11:30:09
time.07/05/16.11:30:10
time.07/05/16.11:30:12
time.07/05/16.11:30:16
time.07/05/16.11:30:20
time.07/05/16.11:30:24
time.07/05/16.11:30:28
time.07/05/16.11:30:32
^C
tiian@ubuntu1004:~$
~~~

### Terminal 2 output

~~~
tiian@ubuntu1004:~$ /tmp/sh2 
time.07/05/16.11:30:11
time.07/05/16.11:30:14
time.07/05/16.11:30:18
time.07/05/16.11:30:22
time.07/05/16.11:30:26
time.07/05/16.11:30:30
time.07/05/16.11:30:34
^C
tiian@ubuntu1004:~$
~~~

### Terminal 3 output

~~~
tiian@ubuntu1004:~$ /tmp/sh2 
time.07/05/16.11:30:13
time.07/05/16.11:30:17
time.07/05/16.11:30:21
time.07/05/16.11:30:25
time.07/05/16.11:30:29
time.07/05/16.11:30:33
time.07/05/16.11:30:36
time.07/05/16.11:30:38
^C
tiian@ubuntu1004:~$
~~~

### Terminal 4 output

~~~
tiian@ubuntu1004:~$ /tmp/sh2 
time.07/05/16.11:30:15
time.07/05/16.11:30:19
time.07/05/16.11:30:23
time.07/05/16.11:30:27
time.07/05/16.11:30:31
time.07/05/16.11:30:35
time.07/05/16.11:30:37
time.07/05/16.11:30:39
^C
tiian@ubuntu1004:~$
~~~

**Add a sub second**, fraction of a second conversion specification to the format, change */tmp/sh2* script as below:

~~~
tiian@ubuntu1004:~$ cat /tmp/sh2 
#!/bin/sh
while (true)
do
	flom -i 10000 -r "_t_time.%D.%T#f[10]" -- echo
done
tiian@ubuntu1004:~$
~~~

Execute again the script in the 4 terminals... and verify by yourself that **no duplicated timestamp** is provided by FLoM to the lockers.

Changing the script again, you can get an idea about how many timestamps per second you can generate using FLoM command line:

~~~
tiian@ubuntu1004:~$ cat /tmp/sh2 
#!/bin/sh
while (true)
do
	flom -i 10000 -r "_t_time.%D.%T#ffffff[10]" -- echo
done
tiian@ubuntu1004:~$
~~~

### Terminal output

~~~
tiian@ubuntu1004:~$ /tmp/sh2 
time.07/05/16.11:43:59.896944
time.07/05/16.11:43:59.899419
time.07/05/16.11:43:59.901677
time.07/05/16.11:43:59.904002
time.07/05/16.11:43:59.906439
time.07/05/16.11:43:59.908720
time.07/05/16.11:43:59.911056
time.07/05/16.11:43:59.913345
time.07/05/16.11:43:59.915782
time.07/05/16.11:43:59.918470
time.07/05/16.11:43:59.921006
time.07/05/16.11:43:59.923200
time.07/05/16.11:43:59.925509
time.07/05/16.11:43:59.927920
time.07/05/16.11:43:59.930137
time.07/05/16.11:43:59.932476
time.07/05/16.11:43:59.934852
time.07/05/16.11:43:59.937385
time.07/05/16.11:43:59.939671
time.07/05/16.11:43:59.942071
time.07/05/16.11:43:59.944415
time.07/05/16.11:43:59.946561
time.07/05/16.11:43:59.948938
time.07/05/16.11:43:59.951316
time.07/05/16.11:43:59.953583
time.07/05/16.11:43:59.956113
time.07/05/16.11:43:59.958598
time.07/05/16.11:43:59.960878
time.07/05/16.11:43:59.963227
time.07/05/16.11:43:59.965872
time.07/05/16.11:43:59.968543
time.07/05/16.11:43:59.971140
time.07/05/16.11:43:59.973543
time.07/05/16.11:43:59.975912
time.07/05/16.11:43:59.978111
time.07/05/16.11:43:59.980560
time.07/05/16.11:43:59.983045
time.07/05/16.11:43:59.985340
time.07/05/16.11:43:59.987775
time.07/05/16.11:43:59.990205
time.07/05/16.11:43:59.992579
time.07/05/16.11:43:59.995013
time.07/05/16.11:43:59.997420
time.07/05/16.11:44:00.000004
time.07/05/16.11:44:00.002414
time.07/05/16.11:44:00.004932
time.07/05/16.11:44:00.007446
time.07/05/16.11:44:00.009651
time.07/05/16.11:44:00.012016
time.07/05/16.11:44:00.014486
time.07/05/16.11:44:00.016864
time.07/05/16.11:44:00.019414
time.07/05/16.11:44:00.021879
time.07/05/16.11:44:00.024319
time.07/05/16.11:44:00.026739
time.07/05/16.11:44:00.029039
time.07/05/16.11:44:00.031560
time.07/05/16.11:44:00.033855
time.07/05/16.11:44:00.036230
time.07/05/16.11:44:00.038681
time.07/05/16.11:44:00.041013
time.07/05/16.11:44:00.043305
time.07/05/16.11:44:00.045797
time.07/05/16.11:44:00.048069
time.07/05/16.11:44:00.050326
time.07/05/16.11:44:00.052746
time.07/05/16.11:44:00.055178
time.07/05/16.11:44:00.057455
time.07/05/16.11:44:00.059814
time.07/05/16.11:44:00.062201
time.07/05/16.11:44:00.064555
time.07/05/16.11:44:00.067002
time.07/05/16.11:44:00.069589
time.07/05/16.11:44:00.071900
time.07/05/16.11:44:00.074362
time.07/05/16.11:44:00.076794
time.07/05/16.11:44:00.079331
time.07/05/16.11:44:00.081731
time.07/05/16.11:44:00.084032
time.07/05/16.11:44:00.086462
time.07/05/16.11:44:00.088802
time.07/05/16.11:44:00.091058
time.07/05/16.11:44:00.093412
time.07/05/16.11:44:00.095839
time.07/05/16.11:44:00.098607
time.07/05/16.11:44:00.101035
time.07/05/16.11:44:00.103359
time.07/05/16.11:44:00.105838
time.07/05/16.11:44:00.108178
time.07/05/16.11:44:00.110416
time.07/05/16.11:44:00.112818
time.07/05/16.11:44:00.115209
time.07/05/16.11:44:00.117490
time.07/05/16.11:44:00.119930
time.07/05/16.11:44:00.122339
time.07/05/16.11:44:00.124562
time.07/05/16.11:44:00.127037
time.07/05/16.11:44:00.129465
time.07/05/16.11:44:00.132204
time.07/05/16.11:44:00.134769
time.07/05/16.11:44:00.137333
time.07/05/16.11:44:00.139697
time.07/05/16.11:44:00.141910
time.07/05/16.11:44:00.144343
time.07/05/16.11:44:00.146848
time.07/05/16.11:44:00.149190
time.07/05/16.11:44:00.151590
time.07/05/16.11:44:00.153906
time.07/05/16.11:44:00.156195
time.07/05/16.11:44:00.158554
time.07/05/16.11:44:00.160982
time.07/05/16.11:44:00.163506
time.07/05/16.11:44:00.166284
time.07/05/16.11:44:00.168990
time.07/05/16.11:44:00.171401
time.07/05/16.11:44:00.173944
time.07/05/16.11:44:00.176430
time.07/05/16.11:44:00.178903
time.07/05/16.11:44:00.181338
time.07/05/16.11:44:00.183672
time.07/05/16.11:44:00.186106
time.07/05/16.11:44:00.188431
time.07/05/16.11:44:00.190669
time.07/05/16.11:44:00.193048
time.07/05/16.11:44:00.195536
time.07/05/16.11:44:00.198021
time.07/05/16.11:44:00.200551
time.07/05/16.11:44:00.202823
time.07/05/16.11:44:00.205210
time.07/05/16.11:44:00.207674
time.07/05/16.11:44:00.209999
time.07/05/16.11:44:00.212419
time.07/05/16.11:44:00.214663
time.07/05/16.11:44:00.216883
time.07/05/16.11:44:00.219334
time.07/05/16.11:44:00.221747
time.07/05/16.11:44:00.224027
time.07/05/16.11:44:00.226240
time.07/05/16.11:44:00.228606
time.07/05/16.11:44:00.231251
time.07/05/16.11:44:00.234026
time.07/05/16.11:44:00.236319
time.07/05/16.11:44:00.238607
time.07/05/16.11:44:00.241025
^C
tiian@ubuntu1004:~$
~~~

Taking a look to the above data, the minimum difference in the millisecond value is about 2 ms: the value is in accordance with the synchronization time measured before.    
With a longer execution, not shown here for brevity, I have measured 429 distinct timestamps (and locks) per second using my virtual environment.    
Using [libflom](../libflom/libflom.md) (FLoM API), from your own custom program, could improve the performance.

## Use cases
**How can timestamp resources be used?**

* you need to synchronize a bunch of commands/scripts and you want that every command/script gets it's own distinct timestamp because you need a *key* with the format and the value of a timestamp
* you need a simple technology to retrieve a distinct timestamp from a command/script in a multiprocessor concurrent environment
* you want to execute exactly *N* commands/scripts and pass them a valid distinct timestamp
