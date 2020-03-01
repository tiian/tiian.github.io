# Use Case 7: distributed command/script synchronization

This use case shows a basic example of synchronization happening between commands running on different systems.

## Before you can start:
Download and install **flom** on two different systems: they must be able to contact each other using IP network.
I have two systems named "mojan" and "presanella", this is a network check example:

### Terminal 1 (*mojan*) output
    tiian@mojan:/usr$ flom --version
    FLoM: Free Lock Manager
    Copyright (c) 2013-2020, Christian Ferrari; all rights reserved.
    License: GPL (GNU Public License) version 2
    Package name: flom; package version: 1.5.0-dev; release date: 2020-02-25
    TLS supported protocol(s): TLSv1
    Access https://github.com/tiian/flom for project community activities
    Documentation is available at http://www.tiian.org/flom/

    tiian@mojan:/usr$ ping -c 3 presanella
    PING presanella (192.168.1.3) 56(84) bytes of data.
    64 bytes from presanella (192.168.1.3): icmp_seq=1 ttl=64 time=7.30 ms
    64 bytes from presanella (192.168.1.3): icmp_seq=2 ttl=64 time=7.23 ms
    64 bytes from presanella (192.168.1.3): icmp_seq=3 ttl=64 time=7.92 ms
    
    --- presanella ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 7.230/7.486/7.921/0.309 ms
    tiian@mojan:/usr$

### Terminal 2 (*presanella*) output
    tiian@presanella:/usr$ flom --version
    FLoM: Free Lock Manager
    Copyright (c) 2013-2020, Christian Ferrari; all rights reserved.
    License: GPL (GNU Public License) version 2
    Package name: flom; package version: 1.5.0-dev; release date: 2020-02-25
    TLS supported protocol(s): TLSv1
    Access https://github.com/tiian/flom for project community activities
    Documentation is available at http://www.tiian.org/flom/

    tiian@presanella:/usr$ ping -c 3 mojan
    PING mojan (192.168.1.4) 56(84) bytes of data.
    64 bytes from mojan (192.168.1.4): icmp_req=1 ttl=64 time=73.4 ms
    64 bytes from mojan (192.168.1.4): icmp_req=2 ttl=64 time=95.3 ms
    64 bytes from mojan (192.168.1.4): icmp_req=3 ttl=64 time=116 ms
    
    --- mojan ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 73.458/95.202/116.794/17.693 ms
    tiian@presanella:/usr$

If anything looks good, you can go on with the experiment...

## Activate a flom network daemon and check it's up & running:
System *mojan* will be our *flom network daemon* (server) system, so we have to activate it with the following commands:

    tiian@mojan:/usr$ pgrep flom
    tiian@mojan:/usr$ flom -a 192.168.1.4 -d -1 -- true
    tiian@mojan:/usr$ pgrep flom
    2198
    tiian@mojan:/usr$ 
There was no a running daemon before we started it, now there's exactly one *flom* running process with *process id 2198*.

Check the daemon is serving local requests:

    tiian@mojan:/usr$ flom -a 192.168.1.4 -d 0 -- ls
    bin  games  include  lib  lib64  local	sbin  share  src
    tiian@mojan:/usr$

Switch to the second terminal (second system) and try the same command using the IP address or the network name of *flom daemon*:

    tiian@presanella:/usr$ flom -a 192.168.1.4 -d 0 -- ls
    bin  games  include  lib  local  sbin  share  src
    tiian@presanella:/usr$ flom -a mojan -d 0 -- ls
    bin  games  include  lib  local  sbin  share  src
    tiian@presanella:/usr$

## Experiment synchronization:
1. inside the first terminal write this command at prompt, but do **not** press "enter": "**flom -a mojan -d 0 \-\- ls**"
2. inside the second terminal write this command at prompt: "**flom -a mojan -d 0 \-\- sleep 10**"
3. now press "enter" key at the second terminal (where you have written "*flom -a mojan -d 0 \-\- sleep 10*")
4. switch to first terminal and press "enter" key

## Expected result:
1. the second terminal (system *presanella* in the above example) pauses for 10 seconds
2. the first terminal (system *mojan* in the above example) pauses and displays the output of command "**ls**" after the second terminal *sleeping* terminates

![](use_case_1_5b_6b_7_8_9_14.png)

### Terminal 1 output:
    tiian@mojan:/usr$ flom -a 192.168.1.4 -d 0 -- ls
    bin  games  include  lib  lib64  local	sbin  share  src
    tiian@mojan:/usr$

### Terminal 2 output:
    tiian@presanella:/usr$ flom -a mojan -d 0 -- sleep 10
    tiian@presanella:/usr$

## Explanation:
command "**sleep 10**" and command "**ls**" synchronized: "**ls**" executed after "**sleep 10**" completion.
*flom* command protects (synchronizes) the execution of the command (or script) specified after the *\-\-* separator on the command line.

## Special attention:
Pay attention to local names resolution: many times they point to loopback address.
See the following examples

### How names are resolved by *mojan* system:
    tiian@mojan:/usr$ ping -c 1 mojan
    PING mojan (127.0.1.1) 56(84) bytes of data.
    64 bytes from mojan (127.0.1.1): icmp_seq=1 ttl=64 time=0.050 ms
    
    --- mojan ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.050/0.050/0.050/0.000 ms
    tiian@mojan:/usr$ ping -c 1 presanella
    PING presanella (192.168.1.3) 56(84) bytes of data.
    64 bytes from presanella (192.168.1.3): icmp_seq=1 ttl=64 time=3.31 ms
    
    --- presanella ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 3.319/3.319/3.319/0.000 ms
    tiian@mojan:/usr$

### How the same names are resolved by *presanella* system:
    tiian@presanella:/usr$ ping -c 1 mojan
    PING mojan (192.168.1.4) 56(84) bytes of data.
    64 bytes from mojan (192.168.1.4): icmp_req=1 ttl=64 time=90.6 ms
    
    --- mojan ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 90.642/90.642/90.642/0.000 ms
    tiian@presanella:/usr$ ping -c 1 presanella
    PING presanella (127.0.1.1) 56(84) bytes of data.
    64 bytes from presanella (127.0.1.1): icmp_req=1 ttl=64 time=0.102 ms
    
    --- presanella ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.102/0.102/0.102/0.000 ms
    tiian@presanella:/usr$

This is a common behavior and can lead to unexpected result:
1. activating a *daemon* using its network name is equivalent to activate the daemon on address 127.0.0.1 (loopback): it can be reached only by processes inside the same system
2. trying to reach a *daemon* using its network name from the same system hosting the *daemon* may not work if the *daemon* was not started using network name.

## Extra considerations related to -d option:
*-d* (*\-\-daemon-lifespan*) is the parameter used to specify how long a *flom daemon* must run.   
-d -1 is used in the above example with the meaning "start a daemon and don't stop it"   
-d 0 is used in the above example with the meaning "don't start a daemon"   
First usage is typically *server oriented*, while the second usage is typically *client oriented*.   
If you are not happy to specify "-d 0" for every *client* command, you can set *Lifespan* option in a [configuration](../Configuration.md) file.

To terminate a running *flom daemon*, simply use *kill* or *pkill* command:

    tiian@mojan:/usr$ pgrep flom
    2198
    tiian@mojan:/usr$ pkill flom
    tiian@mojan:/usr$ pgrep flom
    tiian@mojan:/usr$

## Summary
This simple usage form of command *flom* allows you to synchronize commands/scripts running on different systems.
