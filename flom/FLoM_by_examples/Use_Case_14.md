# Use Case 14: use hierarchical resources to lock files and directories

FLoM is not expressly designed to lock files inside a filesystem, as listed in [similar tools](../Similar_Tools.md) there are many alternative tools that can be used to solve the *file locking issues*.   
If, for any reason, you prefer to use FLoM instead some other tool, *simple resources* are not a practical way because a *simple resource* name can contain only alphabetic and numeric characters while a filename typically contains characters like dots, commas, slashes, underscores and so on; you could make some shell hacks like applying **md5sum** to filename and adding an alphabetic char in front of the exadecimal MD5 signature, but one of the purposes of FLoM is reducing shell scripting hacks in the lock playfield.   
*Hierarchical resources* can help you because they accept any character inside the name.

## Open two terminals and try this experiment:

1. inside the first terminal write this command at prompt, but do **not** press "enter": "**flom -r /foo/bar \-\- ls**"
2. inside the second terminal write this command at prompt, but do **not** press "enter": "**flom -r /foo/bar \-\- ping -c 5 localhost**"
3. now press "enter" key at the second terminal
4. switch to first terminal and press "enter" key

### Expected result:

1. the second terminal pings *localhost* five times
2. the first terminal displays the current directory content after first terminal ended pinging

![](use_case_1_5b_6b_7_8_9_14.png)

#### Terminal 2 output:

    tiian@mojan:/usr$ flom -r /foo/bar -- ping -c 5 localhost
    PING localhost (127.0.0.1) 56(84) bytes of data.
    64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.047 ms
    64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.042 ms
    64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.039 ms
    64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.048 ms
    64 bytes from localhost (127.0.0.1): icmp_seq=5 ttl=64 time=0.049 ms
    
    --- localhost ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3997ms
    rtt min/avg/max/mdev = 0.039/0.045/0.049/0.003 ms
    tiian@mojan:/usr$

#### Terminal 1 output:

    tiian@mojan:/usr$ flom -r /foo/bar -- ls
    bin  games  include  lib  lib64  local	sbin  share  src
    tiian@mojan:/usr$

### Explanation:
the second command "**ping -c 5 localhost**" locks the resource "/foo/bar"; the first command "**ls -la**" enqueues and waits the resource "/foo/bar" becomes free before starting.

## Some hierarchical resource details

### Naming convention
The name of an hierarchical resource must comply with these restrictions:
1. *hierarchical resource* names **must** start with a *slash* ("/")
2. *hierarchical resource* names can **not** end with a *slash* ("/")
These are **valid** names:

* /tmp/foo.txt
* /foo/bar
* /this/is/my/file

These are **not** valid names:

* foo.txt   (it does **not** start with a *slash*)
* /foo/bar/   (it **does** end with a *slash*)
 
Which characters you use inside a *hierarchical resource* name is up to you, you only have to comply with *slashes restrictions* explained above.

### Supported options
Hierarchical resources support these options:

* *-o, \-\-resource-timeout*
* *-l, \-\-lock-mode*

Hierarchical resources don't support this option:

* *-q, \-\-resource-quantity*

### Why should I use FLoM to manage file locks?
As previously stated, other tools exists to manage file locks. So, why should I use FLOM instead of something else?

#### Some tools creates persistent locks
Persistent locks can be desiderable if you want a lock to survive an application or a system crash. Persistent locks can be a source of issues if your use case does not allow you to easily answer the question: *"Is this persistent lock really necessary or is it a lock I should delete?"*

#### Some lock technologies are not safe if the filesystem is not local
Notably, some NFS versions did not safely manage the file locks.

#### Some (distributed) filesystems do not have lock primitives
Non persistent file locks need file locks primitives at the filesystem level: some does support locking APIs.

#### I already use FLoM...
FLoM allows you to manage distintc type of resources: you could choose FLoM just because you can use with files too.

## Summary
This use case explains you how to use a file name as a resource name when using FLoM.

### See also
FLoM available arguments are documented in man page: use **man flom**.
FLoM [configuration](../Configuration.md) explains how you can specify flom behavior without using command line arguments.
