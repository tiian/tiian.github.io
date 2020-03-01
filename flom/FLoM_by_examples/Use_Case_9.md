# Use Case #9: distributed command/script synchronization, dynamic network mode (cloud friendly)

This use case shows extends [Use Case 8](Use_Case_8.md) and shows as auto-discovery feature works.

## Before you can start:
Download and install **flom** on two different systems: they must be able to contact each other using IP network; it is suggested to play with [Use Case 7](Use_Case_7.md) and [Use Case 8](Use_Case_8.md) before trying this one.

### UDP/IP multicast and firewalls
Please pay attention some Linux distros configure a default firewall that silently drop multicast datagrams: disable your firewall or configure it to **not** drop multicast datagrams.   
**openSUSE** 12.2 is such a distribution.

## Activate a flom network daemon and check it's up & running:
System *mojan* will be our *flom network daemon* (server) system, so we have to activate it with the following commands:

    tiian@mojan:/usr$ pgrep flom
    tiian@mojan:/usr$ flom -A 239.255.0.1 -d -1 -- true
    tiian@mojan:/usr$ pgrep flom
    2634
    tiian@mojan:/usr$ ps -ef | grep flom | grep -v grep
    tiian     2695     1  0 22:09 ?        00:00:00 flom -A 239.255.0.1 -d -1 -- true
    tiian@mojan:/usr$

There was no a running daemon before we started it, now there's exactly one *flom* running process with *process id 2695*.

Check the daemon is serving local requests using auto-discovery feature (UDP/IP multicast):

    tiian@mojan:/usr$ flom -A 239.255.0.1 -d 0 -- ls
    bin  games  include  lib  lib64  local	sbin  share  src
    tiian@mojan:/usr$

Switch to the second terminal (second system) and try the same command using the auto-discovery feature:

    tiian@presanella:/usr$ flom -A 239.255.0.1 -d 0 -- ls
    bin  games  include  lib  local  sbin  share  src
    tiian@presanella:/usr$

## Experiment synchronization:
1. inside the first terminal write this command at prompt, but do **not** press "enter": "**flom -A 239.255.0.1 -d 0 \-\- ls**"
2. inside the second terminal write this command at prompt: "**flom -A 239.255.0.1 -d 0 \-\- sleep 10**"
3. now press "enter" key at the second terminal (where you have written "*flom -A 239.255.0.1 -d 0 \-\- sleep 10*")
4. switch to first terminal and press "enter" key

## Expected result:
1. the second terminal (system *presanella* in the above example) pauses for 10 seconds
2. the first terminal (system *mojan* in the above example) pauses and displays the output of command "**ls**" after the second terminal *sleeping* terminates

![](use_case_1_5b_6b_7_8_9_14.png)

### Terminal 1 output:
    tiian@mojan:/usr$ flom -A 239.255.0.1 -d 0 -- ls
    bin  games  include  lib  lib64  local	sbin  share  src
    tiian@mojan:/usr$

### Terminal 2 output:
    tiian@presanella:/usr$ flom -A 239.255.0.1 -d 0 -- sleep 10
    tiian@presanella:/usr$

## Explanation:
command "**sleep 10**" and command "**ls**" synchronized: "**ls**" executed after "**sleep 10**" completion.
*flom* command protects (synchronizes) the execution of the command (or script) specified after the *\-\-* separator on the command line.

### Dynamic network mode explanation:
*flom daemon* has been activated on the first system: it automatically choose a valid IP address and port; using *netstat* command we can see the sockets opened by *flom daemon*:

    tiian@mojan:/usr$ netstat -punta|grep flom
    tcp        0      0 0.0.0.0:48210           0.0.0.0:*               LISTEN      2695/flom       
    udp        0      0 239.255.0.1:28015       0.0.0.0:*                           2695/flom       
    tiian@mojan:/usr$ 

The first row means that *flom daemon* is listening an all IP interfaces (address "0.0.0.0") for TCP connections on port 48210.   
The second row means that *flom daemon* subscribed to multicast group (UDP/IP) 239.255.0.1:28015.

## Extra considerations related to -d option:
*-d* (*\-\-daemon-lifespan*) is the parameter used to specify how long a *flom daemon* must run: **using it with dynamic mode, if the network partitions, will lead to a new local undesired daemon**, see also [distributed features](../Distributed_Features.md) page  
-d -1 is used in the above example with the meaning "start a daemon and don't stop it": **using it with dynamic mode, if the network partitions, will lead to a new local undesired daemon**, see also [distributed features](../Distributed_Features.md) page    
-d 0 is used in the above example with the meaning "don't start a daemon"   
First usage is typically *server oriented*, while the second usage is typically *client oriented*.   
If you are not happy to specify "-d 0" for every *client* command, you can set *Lifespan* option in a [configuration](../Configuration.md) file.

## Summary
This example of command *flom* allows you to synchronize commands/scripts running on different systems and you don't have to start *flom daemon* on a specific system: it may be started on any system that can answer to UDP/IP multicast queries.   
With dynamic network mode all the information needed to start and reach the *flom daemon* is the UDP/IP multicast address and port (*multicast group*): this can save you a lot of configuration pains, but exposes you to serious issues in case of network partitions (see [distributed features](../Distributed_Features.md) for details).   
The **dynamic mode** is *cloud friendly* because a pool of systems can elige automatically a *daemon* without a *special node*.
