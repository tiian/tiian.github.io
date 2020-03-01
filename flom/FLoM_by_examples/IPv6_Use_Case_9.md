# IPv6 Use Case #9: distributed command/script synchronization, dynamic network mode (cloud friendly)

This use case shows extends [Use Case 9](Use_Case_9.md) and shows as dynamic network mode feature works.

## Before you can start:
Download and install **flom** on two different systems: they must be able to contact each other using IP network; it is suggested to play with [Use Case 7](Use_Case_7.md) and [Use Case 8](Use_Case_8.md) before trying this one.

### UDP/IP multicast and firewalls
Please pay attention some Linux distros configure a default firewall that silently drop multicast datagrams: disable your firewall or configure it to **not** drop multicast datagrams.   
**openSUSE** 12.2 is such a distribution.

If you did not already try [IPv6 Use Case 8](IPv6_Use_Case_8.md), you should take a look to it, especially to the paragraph that explains how to verify there are not system and network issues related to UDP/IPv6 datagrams.

## Activate a flom network daemon and check it's up & running:
System *centos71-64* will be our *flom network daemon* (server) system, so we have to activate it with the following commands:

    [tiian@centos71-64 usr]$ pgrep flom
    [tiian@centos71-64 usr]$ flom -A ff02::1 -d -1 -- true
    [tiian@centos71-64 usr]$ pgrep flom
    2025
    [tiian@centos71-64 usr]$ ps -ef | grep flom | grep -v grep
    tiian     2025     1  0 21:50 ?        00:00:00 flom -A ff02::1 -d -1 -- true
    [tiian@centos71-64 usr]$

There was no a running daemon before we started it, now there's exactly one *flom* running process with *process id 2025*.

Check the daemon is serving local requests using auto-discovery feature (UDP/IP multicast):

    [tiian@centos71-64 usr]$ flom -A ff02::1 -d 0 -- ls
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    [tiian@centos71-64 usr]$

Switch to the second terminal (second system) and try the same command using the auto-discovery feature:

    tiian@ubuntu1404-64:/usr$ flom -A ff02::1 -d 0 -- ls
    bin  games  include  lib  local  sbin  share  src
    tiian@ubuntu1404-64:/usr$

## Summary
This example of command *flom* allows you to synchronize commands/scripts running on different systems and you don't have to start *flom daemon* on a specific system: it may be started on any system that can answer to UDP/IPv6 multicast queries.   
With dynamic network mode all the information needed to start and reach the *flom daemon* is the UDP/IPv6 multicast address and port (*multicast group*): this can save you a lot of configuration pains, but exposes you to serious issues in case of network partitions (see [distributed features](../Distributed_Features.md) for details).   
The **dynamic mode** is *cloud friendly* because a pool of systems can automatically elige a *daemon* without a *special node*.
