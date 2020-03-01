# IPv6 Use Case 7: distributed command/script synchronization

This use case shows a basic example of synchronization happening between commands running on different systems: this is basically the same use case shown in [Use Case 7](Use_Case_7.md), but with IPv6 instead of IPv4 network connectivity.

## Before you can start:
Download and install **flom** on two different systems: they must be able to contact each other using IP network.   
I have two systems named "mojan" and "presanella", this is a network check example:

### Terminal 1 (*ubuntu1404-64*) output
    
    tiian@ubuntu1404-64:/tmp$ flom --version
    FLoM: Free LOck Manager
    Copyright (c) 2013-2015, Christian Ferrari; all rights reserved.
    License: GPL (GNU Public License) version 2
    Package name: flom; package version: 1.1.1; release date: 2015-11-26
    Access http://sourceforge.net/projects/flom/ for project community activities
    tiian@ubuntu1404-64:/tmp$

    tiian@ubuntu1404-64:/tmp$ ping6 -c 3 centos71-64.brenta.org -I eth0
    PING centos71-64.brenta.org(fe80::5054:ff:fe5f:6392) from fe80::5054:ff:fe18:78bd eth0: 56 data bytes
    64 bytes from fe80::5054:ff:fe5f:6392: icmp_seq=1 ttl=64 time=0.273 ms
    64 bytes from fe80::5054:ff:fe5f:6392: icmp_seq=2 ttl=64 time=0.553 ms
    64 bytes from fe80::5054:ff:fe5f:6392: icmp_seq=3 ttl=64 time=0.358 ms
    
    --- centos71-64.brenta.org ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 0.273/0.394/0.553/0.119 ms
    tiian@ubuntu1404-64:/tmp$

### Terminal 2 (*centos71-64*) output
    
    [tiian@centos71-64 tmp]$ flom --version
    FLoM: Free LOck Manager
    Copyright (c) 2013-2015, Christian Ferrari; all rights reserved.
    License: GPL (GNU Public License) version 2
    Package name: flom; package version: 1.1.1; release date: 2015-11-26
    Access http://sourceforge.net/projects/flom/ for project community activities
    [tiian@centos71-64 tmp]$

    [tiian@centos71-64 tmp]$ ping -c 3 ubuntu1404-64.brenta.org -I eth0
    PING ubuntu1404-64.brenta.org (192.168.122.212) from 192.168.122.12 eth0: 56(84) bytes of data.
    64 bytes from ubuntu1404-64.brenta.org.122.168.192.in-addr.arpa (192.168.122.212): icmp_seq=1 ttl=64 time=0.206 ms
    64 bytes from ubuntu1404-64.brenta.org.122.168.192.in-addr.arpa (192.168.122.212): icmp_seq=2 ttl=64 time=0.249 ms
    64 bytes from ubuntu1404-64.brenta.org.122.168.192.in-addr.arpa (192.168.122.212): icmp_seq=3 ttl=64 time=0.190 ms
    
    --- ubuntu1404-64.brenta.org ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2000ms
    rtt min/avg/max/mdev = 0.190/0.215/0.249/0.024 ms
    [tiian@centos71-64 tmp]$

If anything looks good, you can go on with the experiment...
**Note:** please pay attention that with IPv6 addresses you must use "-I" option followed by the interface that must be used to reach the destination.

## Activate a flom network daemon and check it's up & running:
System *ubuntu1404-64* will be our *flom network daemon* (server) system, so we have to activate it with the following commands:

    tiian@ubuntu1404-64:/tmp$ pgrep flom
    tiian@ubuntu1404-64:/tmp$ flom -a ubuntu1404-64.brenta.org -n eth0 -d -1 -- truetiian@ubuntu1404-64:/tmp$ pgrep flom
    12340
    tiian@ubuntu1404-64:/tmp$

There was no a running daemon before we started it, now there's exactly one *flom* running process with *process id 12340*.

Check the daemon is serving local requests:

    tiian@ubuntu1404-64:/tmp$ flom -a ubuntu1404-64.brenta.org -n eth0 -d -0 -- ls
    flom-1.1.1  flom-1.1.1.tar.gz  hsperfdata_tiian
    tiian@ubuntu1404-64:/tmp$

you can also check the listening port (28015) and address (fe80::5054:ff:fe1):

    tiian@ubuntu1404-64:/tmp$ netstat -unta
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
    tcp        0      0 192.168.122.212:22      192.168.122.1:58120     ESTABLISHED
    tcp6       0      0 fe80::5054:ff:fe1:28015 :::*                    LISTEN     
    tcp6       0      0 :::22                   :::*                    LISTEN     
    udp        0      0 0.0.0.0:68              0.0.0.0:*                          
    udp        0      0 0.0.0.0:41328           0.0.0.0:*                          
    udp6       0      0 :::34575                :::*                               
    tiian@ubuntu1404-64:/tmp$

Switch to the second terminal (second system) and try the same command using the IP address or the network name of *flom daemon*:

    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org -n eth0 -d 0 -- ls
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    [tiian@centos71-64 usr]$

## Special considerations related IPv6: ##
IPv6 networking needs, at least when the operating system is Linux, to specify the network interface.
Here is an example of what happens if you miss it:

    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org -d 0 -- ls
    flom_client_connect: ret_cod=-104 (ERROR: 'connect' function returned an error condition)
    [tiian@centos71-64 usr]$

activating the trace some more information can be retrieved:

    [tiian@centos71-64 usr]$ export FLOM_TRACE_MASK=0x8
    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org -d 0 -- ls
    2015-11-26 22:01:12.735308 [13160/0x14b2000] flom_client_connect
    2015-11-26 22:01:12.735434 [13160/0x14b2000] flom_client_connect_tcp
    2015-11-26 22:01:12.735443 [13160/0x14b2000] flom_client_connect_tcp: connecting to address 'ubuntu1404-64.brenta.org' and port 28015
    2015-11-26 22:01:12.737057 [13160/0x14b2000] flom_client_connect_tcp/getaddrinfo(): [ai_flags=2,ai_family=2,ai_socktype=1,ai_protocol=6,ai_addrlen=16,ai_canonname='ubuntu1404-64.brenta.org'] [ai_flags=2,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='{null}'] 
    2015-11-26 22:01:12.737100 [13160/0x14b2000] flom_client_connect_tcp_try: sa addrlen=16; IPv4 address:, sin_port=28015, sin_addr='192.168.122.212'
    2015-11-26 22:01:12.738327 [13160/0x14b2000] flom_client_connect_tcp_try: sa addrlen=16; IPv4 address:, sin_port=28015, sin_addr='192.168.122.212'
    2015-11-26 22:01:12.738731 [13160/0x14b2000] flom_client_connect_tcp_try/connect(): errno=111 'Connection refused', skipping...
    2015-11-26 22:01:12.738769 [13160/0x14b2000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=0
    2015-11-26 22:01:12.738829 [13160/0x14b2000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=0
    2015-11-26 22:01:12.738858 [13160/0x14b2000] flom_client_connect_tcp_try/connect(): errno=22 'Invalid argument', skipping...
    2015-11-26 22:01:12.738871 [13160/0x14b2000] flom_client_connect_tcp: connection failed, a new daemon can not be started because daemon lifespan is 0
    2015-11-26 22:01:12.738886 [13160/0x14b2000] flom_client_connect_tcp/excp=3/ret_cod=-104/errno=22
    2015-11-26 22:01:12.738894 [13160/0x14b2000] flom_client_connect/excp=1/ret_cod=-104/errno=22
    flom_client_connect: ret_cod=-104 (ERROR: 'connect' function returned an error condition)
    [tiian@centos71-64 usr]$


the critical point is this one:

    2015-11-26 22:01:12.738829 [13160/0x14b2000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=0

FLoM does not specify the interface that must be used ("sin6_scope_id=0") and the kernel is not able to choose the right interface to reach the destination.   
Compare the previous output with the correct execution:

    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org -n eth0 -d 0 -- ls
    2015-11-26 22:03:14.192255 [13161/0x17b4000] flom_client_connect
    2015-11-26 22:03:14.192288 [13161/0x17b4000] flom_client_connect_tcp
    2015-11-26 22:03:14.192291 [13161/0x17b4000] flom_client_connect_tcp: connecting to address 'ubuntu1404-64.brenta.org' and port 28015
    2015-11-26 22:03:14.192789 [13161/0x17b4000] flom_client_connect_tcp/getaddrinfo(): [ai_flags=2,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='ubuntu1404-64.brenta.org'] 
    2015-11-26 22:03:14.192798 [13161/0x17b4000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=0
    2015-11-26 22:03:14.192806 [13161/0x17b4000] flom_client_connect_tcp_try: overriding field sin6_scope_id with value 2
    2015-11-26 22:03:14.192819 [13161/0x17b4000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=2
    2015-11-26 22:03:14.193217 [13161/0x17b4000] flom_client_connect_tcp/excp=7/ret_cod=0/errno=0
    2015-11-26 22:03:14.193223 [13161/0x17b4000] flom_client_connect: set FD_CLOEXEC to fd=3
    2015-11-26 22:03:14.193225 [13161/0x17b4000] flom_client_connect/excp=9/ret_cod=0/errno=0
    2015-11-26 22:03:14.193227 [13161/0x17b4000] flom_client_lock
    2015-11-26 22:03:14.193552 [13161/0x17b4000] flom_client_lock/excp=19/ret_cod=0/errno=0
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    2015-11-26 22:03:14.194507 [13161/0x17b4000] flom_client_unlock
    2015-11-26 22:03:14.194527 [13161/0x17b4000] flom_client_unlock/excp=4/ret_cod=0/errno=0
    2015-11-26 22:03:14.194531 [13161/0x17b4000] flom_client_disconnect
    2015-11-26 22:03:14.194673 [13161/0x17b4000] flom_client_disconnect/shutdown(3,SHUT_RD)=107 ('Transport endpoint is not connected')
    2015-11-26 22:03:14.194681 [13161/0x17b4000] flom_client_disconnect/excp=0/ret_cod=0/errno=107
    [tiian@centos71-64 usr]$

now FLoM is able to set the right network interface:

    2015-11-26 22:03:14.192798 [13161/0x17b4000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=0
    2015-11-26 22:03:14.192806 [13161/0x17b4000] flom_client_connect_tcp_try: overriding field sin6_scope_id with value 2
    2015-11-26 22:03:14.192819 [13161/0x17b4000] flom_client_connect_tcp_try: sa addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe18:78bd', sin6_scope_id=2

and everything goes fine. You can also notify that specifying the option "-d", only the IPv6 address is tried because there's no reason to specify the network interface when using IPv4.

**Note:** instead of "-d *interface*" option you can use the "*address%interface*" notation, but you must be aware that it can **not** be used with IPv6 network aliases because 

###getaddrinfo() does not accept %interface after a network alias:

    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org%eth0 -d 0 -- ls
    2015-11-26 22:07:49.233859 [13163/0x1312000] flom_client_connect
    2015-11-26 22:07:49.233919 [13163/0x1312000] flom_client_connect_tcp
    2015-11-26 22:07:49.233923 [13163/0x1312000] flom_client_connect_tcp: connecting to address 'ubuntu1404-64.brenta.org%eth0' and port 28015
    2015-11-26 22:07:51.514640 [13163/0x1312000] flom_client_connect_tcp/getaddrinfo(): errcode=-2 'Name or service not known'
    2015-11-26 22:07:51.514663 [13163/0x1312000] flom_client_connect_tcp/excp=0/ret_cod=-108/errno=2
    2015-11-26 22:07:51.514668 [13163/0x1312000] flom_client_connect/excp=1/ret_cod=-108/errno=2
    flom_client_connect: ret_cod=-108 (ERROR: 'getaddrinfo' function returned an error condition)
    [tiian@centos71-64 usr]$
    
## Summary
This simple usage form of command *flom* allows you to synchronize commands/scripts running on different systems connected by a IPv6 network.
