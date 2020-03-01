# IPv6 Use Case #8: distributed command/script synchronization, auto-discovery feature

This use case shows extends [Use Case 8](Use_Case_8.md) and shows as auto-discovery feature works with IPv6.

## Before you can start:
Download and install **flom** on two different systems: they must be able to contact each other using IPv6 network; it is suggested to play with [Use Case 7](Use_Case_7.md) before trying this one.

### UDP/IP multicast and firewalls
Please pay attention some Linux distros configure a default firewall that silently drop multicast datagrams: disable your firewall or configure it to **not** drop multicast datagrams.   
**openSUSE** 12.2 is such a distribution.

Other distributions filter some UDP/IPv6 packets; this is a configuration that blocks the autodiscovery feature provided by FLoM:

    [root@centos66-64 tiian]# ip6tables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         
    ACCEPT     all      anywhere             anywhere            state RELATED,ESTABLISHED 
    ACCEPT     ipv6-icmp    anywhere             anywhere            
    ACCEPT     all      anywhere             anywhere            
    ACCEPT     udp      anywhere             fe80::/64           state NEW udp dpt:dhcpv6-client 
    ACCEPT     tcp      anywhere             anywhere            state NEW tcp dpt:ssh 
    REJECT     all      anywhere             anywhere            reject-with icmp6-adm-prohibited 
    
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    REJECT     all      anywhere             anywhere            reject-with icmp6-adm-prohibited 
    
    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination

and this is the *brutal force* removal of the rules:

    [root@centos66-64 tiian]# ip6tables -F
    [root@centos66-64 tiian]# ip6tables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination

**Note:** FLoM is not against network and system security, but this wiki wants to alert you that the autodiscovery feature based on IPv6 multicast can need network and system configuration to properly work.

## Check UDP/IPv6 multicast datagrams flow correctly between your systems
To help the system configuration, FLoM provide a special feature that can be used to assess there are no network and security issues related to UDP/IPv6 datagrams.   
Connect a terminal to the system where you want to execute FLoM *daemon*, and activate a debug listener:

    tiian@ubuntu1404-64:~$ flom --debug-feature=ipv6.multicast.server -A ff02::1
    
connect a terminatl to the system where you want to execute a FLoM *client*, and activate a debug requester:

    [tiian@centos71-64 ~]$ flom --debug-feature=ipv6.multicast.client -A ff02::1

if everything is OK, you should have something like this in the "*server*" terminal:

    tiian@ubuntu1404-64:~$ flom --debug-feature=ipv6.multicast.server -A ff02::1
    Arrived datagram is 'HELLO'
    Sent datagram is 'WELCOME'
    tiian@ubuntu1404-64:~$

and something like this in the "*client*" terminal:

    [tiian@centos71-64 ~]$ flom --debug-feature=ipv6.multicast.client -A ff02::1
    Sent datagram is 'HELLO'
    Arrived datagram is 'WELCOME'
    [tiian@centos71-64 ~]$

This is the algorithm:
1. the client send a multicast datagram ('HELLO') to discover a server
2. the server receive the datagram and replies to the client with another datagram ('WELCOME')

If the previous communication does not work as expected you can activate tracing for a better understanding of the problem:

    tiian@ubuntu1404-64:~$ export FLOM_TRACE_MASK=0x10000
    tiian@ubuntu1404-64:~$ flom --debug-feature=ipv6.multicast.server -A ff02::1
    2015-11-28 21:34:15.655861 [1040/0x1133000] flom_debug_features
    2015-11-28 21:34:15.655898 [1040/0x1133000] flom_debug_features: name='ipv6.multicast.server'
    2015-11-28 21:34:15.655902 [1040/0x1133000] flom_debug_features_ipv6_multicast_server
    2015-11-28 21:34:15.655905 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: multicast address='ff02::1', multicast port=28015
    2015-11-28 21:34:15.655913 [1040/0x1133000] flom_debug_features_ipv6_multicast_server/getaddrinfo() list pointer 0x1132580
    2015-11-28 21:34:15.655916 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: address returned by getaddrinfo():addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff02::1', sin6_scope_id=0
    2015-11-28 21:34:15.655929 [1040/0x1133000] flom_debug_features_ipv6_multicast_server/getaddrinfo(): [ai_flags=1,ai_family=10,ai_socktype=2,ai_protocol=17,ai_addrlen=28,ai_canonname='{null}'] 
    2015-11-28 21:34:15.655934 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: creating a new socket...
    2015-11-28 21:34:15.655943 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: address returned by getsockname(): addrlen=28; IPv6 address, sin6_port=0, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-28 21:34:15.655951 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: setting SO_REUSEADDR socket property...
    2015-11-28 21:34:15.655954 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: binding to address addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-28 21:34:15.655963 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: address returned by bind() addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-28 21:34:15.655970 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: activating multicast...
    2015-11-28 21:34:15.655976 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: multicast server created, waiting datagram...
    2015-11-28 21:34:30.464295 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: arrived datagram 'HELLO'
    2015-11-28 21:34:30.464321 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: arrived datagram 48 45 4c 4c 4f 
    2015-11-28 21:34:30.464335 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: address returned by recvfrom() addrlen=28; IPv6 address, sin6_port=38661, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:fe5f:6392', sin6_scope_id=2
    Arrived datagram is 'HELLO'
    2015-11-28 21:34:30.464370 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: sending 'WELCOME' to the client...
    2015-11-28 21:34:30.464404 [1040/0x1133000] flom_debug_features_ipv6_multicast_server: sent_bytes=7, welcome_msg_len=7
    Sent datagram is 'WELCOME'
    2015-11-28 21:34:30.464429 [1040/0x1133000] flom_debug_features_ipv6_multicast_server/excp=5/ret_cod=0/errno=2
    2015-11-28 21:34:30.464433 [1040/0x1133000] flom_debug_features/excp=1/ret_cod=0/errno=2
    tiian@ubuntu1404-64:~$

    [tiian@centos71-64 ~]$ export FLOM_TRACE_MASK=0x10000
    [tiian@centos71-64 ~]$ flom --debug-feature=ipv6.multicast.client -A ff02::1
    2015-11-28 21:34:29.004069 [1994/0x1ae7000] flom_debug_features
    2015-11-28 21:34:29.004184 [1994/0x1ae7000] flom_debug_features: name='ipv6.multicast.client'
    2015-11-28 21:34:29.004191 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client
    2015-11-28 21:34:29.004196 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: multicast address='ff02::1', multicast port=28015
    2015-11-28 21:34:29.004206 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client/getaddrinfo() list pointer 0x1ae65a0
    2015-11-28 21:34:29.004210 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: address returned by getaddrinfo():addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff02::1', sin6_scope_id=0
    2015-11-28 21:34:29.004227 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client/getaddrinfo(): [ai_flags=1,ai_family=10,ai_socktype=2,ai_protocol=17,ai_addrlen=28,ai_canonname='{null}'] 
    2015-11-28 21:34:29.004251 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: setting IPV6_MULTICAST_HOPS to value 1
    2015-11-28 21:34:29.004257 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: setting SO_REUSEADDR to value 1
    2015-11-28 21:34:29.004261 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: binding to address: addrlen=28; IPv6 address, sin6_port=0, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-28 21:34:29.004275 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: address returned by bind() addrlen=28; IPv6 address, sin6_port=38661, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-28 21:34:29.004285 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: sending 'HELLO' to the server...
    2015-11-28 21:34:29.004287 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: destination address:addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff02::1', sin6_scope_id=0
    Sent datagram is 'HELLO'
    2015-11-28 21:34:29.004344 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: waiting an answer from the server...
    2015-11-28 21:34:29.004734 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client: received 'WELCOME'
    Arrived datagram is 'WELCOME'
    2015-11-28 21:34:29.004761 [1994/0x1ae7000] flom_debug_features_ipv6_multicast_client/excp=8/ret_cod=0/errno=2
    2015-11-28 21:34:29.004765 [1994/0x1ae7000] flom_debug_features/excp=1/ret_cod=0/errno=2
    [tiian@centos71-64 ~]$

If no error appears at the trace level there's a chance that something in the middle:
* a firewall
* a switch
* a router
drops UDP/IPv6 datagrams.

## Activate a flom network daemon and check it's up & running:
System *ubuntu1404-64* (IPv6 address fe80::5054:ff:fe18:78bd) will be our *flom network daemon* (server) system, so we have to activate it with the following commands:

    tiian@ubuntu1404-64:~$ pgrep flom
    tiian@ubuntu1404-64:~$ flom -a fe80::5054:ff:fe18:78bd -n eth0 -A ff02::1 -d -1 -- true
    tiian@ubuntu1404-64:~$ pgrep flom
    1057
    tiian@ubuntu1404-64:~$ ps -ef | grep flom | grep -v grep
    tiian     1057     1  0 21:38 ?        00:00:00 flom -a fe80::5054:ff:fe18:78bd -n eth0 -A ff02::1 -d -1 -- true
    tiian@ubuntu1404-64:~$

There was no a running daemon before we started it, now there's exactly one *flom* running process with *process id 1057*.

Check the daemon is serving local requests:

    tiian@ubuntu1404-64:/usr$ flom -a fe80::5054:ff:fe18:78bd -n eth0 -d 0 -- ls
    bin  games  include  lib  local  sbin  share  src
    tiian@ubuntu1404-64:/usr$

Switch to the second terminal (second system, IP address 192.168.1.4) and try the same command using the IP address or the network name of *flom daemon*:

    [tiian@centos71-64 usr]$ flom -a fe80::5054:ff:fe18:78bd -n eth0 -d 0 -- ls
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    [tiian@centos71-64 usr]$ flom -a ubuntu1404-64.brenta.org -n eth0 -d 0 -- ls
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    [tiian@centos71-64 usr]$

From the second terminal try the autodiscovery feature based on UDP/IP multicast:

    [tiian@centos71-64 usr]$ flom -A ff02::1 -d 0 -- ls
    bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src	tmp
    [tiian@centos71-64 usr]$

## Summary
This example of command *flom* allows you to synchronize commands/scripts running on different systems and you don't have to start *flom daemon* on a specific system: it may be started on any system that can answer to UDP/IPv6 multicast queries.   
With auto-discovery feature all the information needed to reach the *flom daemon* is the UDP/IPv6 multicast address and port (*multicast group*): this can save you a lot of configuration pains.
