# Observability
Starting from version 1.7.0, FLoM provides an "observability" feature that can be used to collect real time information about what are the *resources* currently managed by a *daemon* and what are the *locks* currently hold by clients. Furthermore, the *enqueued* clients and the *incubating* resources can be inspected.

**Note:** the capability must be considered a *technical preview* because it will require some time to stabilize and make it *fully production ready*.

The *observability* feature provides a Virtual File System (VFS) that mimics the behavior of the well known /proc and /sys virtual filesystems of the Linux operating system. The VFS provided by FLoM is based on [Linux FUSE](https://www.kernel.org/doc/html/next/filesystems/fuse.html) (Filesystem in Userspace) technology and [libfuse](https://github.com/libfuse/libfuse) that are typically available in most Linux distributions.

## Preparing the VFS mount point
The VFS must be mounted somewhere in the same system where FLoM *daemon* is running; for example you can create a mount point inside /tmp:

    ~$ mkdir flom-vfs
    ~$ ls -la flom-vfs/
    total 8
    drwxrwxr-x 2 tiian tiian 4096 dic 31 15:03 .
    drwxrwxrwt 5 root  root  4096 dic 31 15:03 ..

## Activate FLoM daemon with the observability feature
Use **command line option** "-m", "--mount-point-vfs" to activate the feature and use the mount point for the Virtual File System:

    ~$ flom -m /tmp/flom-vfs/ -- sleep 60

and from another session:

    ~$ find /tmp/flom-vfs/
    /tmp/flom-vfs/
    /tmp/flom-vfs/status
    /tmp/flom-vfs/status/incubator
    /tmp/flom-vfs/status/lockers
    /tmp/flom-vfs/status/lockers/3
    /tmp/flom-vfs/status/lockers/3/resource_name
    /tmp/flom-vfs/status/lockers/3/resource_type
    /tmp/flom-vfs/status/lockers/3/holders
    /tmp/flom-vfs/status/lockers/3/holders/2
    /tmp/flom-vfs/status/lockers/3/holders/2/peer_name
    /tmp/flom-vfs/status/lockers/3/holders/2/lock_mode
    /tmp/flom-vfs/status/lockers/3/waitings

when the *daemon* ends, the VFS should be unmounted:

    ~$ find /tmp/flom-vfs/
    /tmp/flom-vfs/

## How to read the provided information
The following examples is intended to provide an overview on the most meaningful information provided by the capability.

Activate the *daemon* using an IP address and without creating the resource:

    $ flom -m /tmp/flom-vfs/ -a 192.168.123.20 -e no -r num[3] -q 2 -- sleep 60

From another session inside the same system, take a look at the VFS:

    ~$ find /tmp/flom-vfs/
    /tmp/flom-vfs/
    /tmp/flom-vfs/status
    /tmp/flom-vfs/status/incubator
    /tmp/flom-vfs/status/incubator/2
    /tmp/flom-vfs/status/incubator/2/peer_name
    /tmp/flom-vfs/status/incubator/2/resource_name
    /tmp/flom-vfs/status/incubator/2/resource_type
    /tmp/flom-vfs/status/lockers

Inspect the content of the files:

    ~$ cat /tmp/flom-vfs/status/incubator/2/peer_name 
    192.168.123.20/57072
    ~$ cat /tmp/flom-vfs/status/incubator/2/resource_name 
    num[3]
    ~$ cat /tmp/flom-vfs/status/incubator/2/resource_type 
    numeric

Modification times are aligned with operations times:

    ~$ ls -la /tmp/flom-vfs/status/incubator/2/
    total 0
    dr-xr-x--- 2 tiian tiian  0 Jan  1 17:18 .
    dr-xr-x--- 2 tiian tiian  0 Jan  1 17:18 ..
    -r--r----- 1 tiian tiian 22 Jan  1 17:18 peer_name
    -r--r----- 1 tiian tiian  7 Jan  1 17:18 resource_name
    -r--r----- 1 tiian tiian  8 Jan  1 17:18 resource_type

The above information can be read as:

* client **2** is connecting from address 192.168.123.20 port 57072
* it's waiting from **numeric** resource of name **num[3]**

From the same system / another system, activate another client that wants to lock the same resource:

    $ flom -a 192.168.123.20 -r num[3] -q 2 -- sleep 300

look at the content of the VFS:

    ~$ find /tmp/flom-vfs/
    /tmp/flom-vfs/
    /tmp/flom-vfs/status
    /tmp/flom-vfs/status/incubator
    /tmp/flom-vfs/status/lockers
    /tmp/flom-vfs/status/lockers/4
    /tmp/flom-vfs/status/lockers/4/resource_name
    /tmp/flom-vfs/status/lockers/4/resource_type
    /tmp/flom-vfs/status/lockers/4/holders
    /tmp/flom-vfs/status/lockers/4/holders/3
    /tmp/flom-vfs/status/lockers/4/holders/3/peer_name
    /tmp/flom-vfs/status/lockers/4/holders/3/quantity
    /tmp/flom-vfs/status/lockers/4/waitings
    /tmp/flom-vfs/status/lockers/4/waitings/2
    /tmp/flom-vfs/status/lockers/4/waitings/2/peer_name
    /tmp/flom-vfs/status/lockers/4/waitings/2/quantity

now the *incubator* is empty because the *daemon* activated the *locker* number **4** for *resource* "num[3]" of *type* "numeric":

    ~$ cat /tmp/flom-vfs/status/lockers/4/resource_name 
    num[3]
    ~$ cat /tmp/flom-vfs/status/lockers/4/resource_type 
    numeric

only *client* number **3** is holding the resource: it's connecting from address **192.168.123.154** and port **58218** and it's holding a *quantity* of **2** for the resource:

    ~$ cat /tmp/flom-vfs/status/lockers/4/holders/3/peer_name 
    192.168.123.154/58218
    ~$ cat /tmp/flom-vfs/status/lockers/4/holders/3/quantity 
    2

*client* number **2** is waiting for the same resource: it's connecting from address **192.168.123.20** and port **57075** and it's waiting for a *quantity* of **2** for the resource:

    ~$ cat /tmp/flom-vfs/status/lockers/4/waitings/2/peer_name 
    192.168.123.20/57075
    tiian@ubuntu1404-64:~$ cat /tmp/flom-vfs/status/lockers/4/waitings/2/quantity 
    2

and, after some time, when the first holder completed, look at it again:

    ~$ find /tmp/flom-vfs/
    /tmp/flom-vfs/
    /tmp/flom-vfs/status
    /tmp/flom-vfs/status/incubator
    /tmp/flom-vfs/status/lockers
    /tmp/flom-vfs/status/lockers/4
    /tmp/flom-vfs/status/lockers/4/resource_name
    /tmp/flom-vfs/status/lockers/4/resource_type
    /tmp/flom-vfs/status/lockers/4/holders
    /tmp/flom-vfs/status/lockers/4/holders/2
    /tmp/flom-vfs/status/lockers/4/holders/2/peer_name
    /tmp/flom-vfs/status/lockers/4/holders/2/quantity
    /tmp/flom-vfs/status/lockers/4/waitings

    ~$ cat /tmp/flom-vfs/status/lockers/4/holders/2/peer_name 
    192.168.123.20/57075
    ~$ cat /tmp/flom-vfs/status/lockers/4/holders/2/quantity 
    2

as expected, the *client* that was waiting the resource is now holding it.

## Troubleshooting
When the FLoM *daemon* exits, the FUSE Virtual File System is automatically unmounted, but under certain conditions, like for example another process that's keeping the filesystem open, the VFS cannot be automatically unmounted. To unmount it manually, two different commands can be used:

    fusermount -u <mountpoint>

    sudo umount -l <mountpoint>

