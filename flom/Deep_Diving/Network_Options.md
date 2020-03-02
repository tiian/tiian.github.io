# Network options
Inside *Network* section of [Configuration](../Configuration.md) files you can activate some non standard options related to TCP/IP keepalive and UDP/IP auto-discovery feature.

## TCP/IP keepalive
Linux kernel allows you to set TCP/IP keepalive parameters on a per socket basis, other *UNIX-like* systems does not.   
Keepalive is useful in many situation, you may read this good article to understand what it is and why it should be used: [TCP Keepalive HOWTO](http://tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO).   
FLoM aggressively uses TCP keepalive to detect disappeared clients: without this protection an abruptely closed system could keep a resource locked for a long time.   
FLoM allows you to specify all the three parameters supplied by keepalive functionality: *time*, *intvl*, *probes*.   
Default values can be easily retrieved running it with verbose mode activated:

    tiian@mojan:~$ flom --verbose
    [...]
    [Network]/TcpKeepaliveTime=60
    [Network]/TcpKeepaliveIntvl=10
    [Network]/TcpKeepaliveProbes=6
    tiian@mojan:~$

These options can be changed using a [Configuration](../Configuration.md) file or using these command line options:

    --tcp-keepalive-time         Local override for SO_KEEPALIVE feature
    --tcp-keepalive-intvl        Local override for SO_KEEPALIVE feature
    --tcp-keepalive-probes       Local override for SO_KEEPALIVE feature

The meaning of the above options is not explained here because I will not be able to make something better than this [document](http://tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO)

## UDP/IP auto-discovery network related options
Inside a [Configuration](../Configuration.md) file you can specify three options that normally have these default values:

    tiian@mojan:~$ flom --verbose
    [...]
    [Network]/DiscoveryAttempts=2
    [Network]/DiscoveryTimeout=500
    [Network]/DiscoveryTTL=1
    tiian@mojan:~$

### DiscoveryAttemps
**Command line option**: "-D", "\-\-discovery-attempts"   
**Meaning**: maximum number of UDP/IP multicast request a FLoM command (client) performs to discover a FLoM daemon (server).   
This example shows what happens if a daemon is not available specifying 7 discovery attempts:

    tiian@mojan:~$ export FLOM_TRACE_MASK=0x8
    tiian@mojan:~$ flom -A 239.255.0.1 -d 0 -D 7 -- true
    2014-03-20 22:13:43.517029 [2623/0x9ffa130] flom_client_connect
    2014-03-20 22:13:43.517274 [2623/0x9ffa130] flom_client_discover_udp
    2014-03-20 22:13:43.517304 [2623/0x9ffa130] flom_client_discover_udp: using address '239.255.0.1' and port 28015
    2014-03-20 22:13:43.517494 [2623/0x9ffa130] flom_client_discover_udp/getaddrinfo(): [ai_flags=2,ai_family=2,ai_socktype=2,ai_protocol=17,ai_addrlen=16,ai_canonname='239.255.0.1'] 
    2014-03-20 22:13:43.517580 [2623/0x9ffa130] flom_client_discover_udp: ai_addr 02 00 6d 6f ef ff 00 01 00 00 00 00 00 00 00 00 
    2014-03-20 22:13:43.517732 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 0...
    2014-03-20 22:13:43.517879 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:44.015698 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:44.015755 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 1...
    2014-03-20 22:13:44.015860 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:44.515686 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:44.515748 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 2...
    2014-03-20 22:13:44.515849 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:45.015690 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:45.015752 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 3...
    2014-03-20 22:13:45.015861 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:45.515691 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:45.515750 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 4...
    2014-03-20 22:13:45.515859 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:46.015692 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:46.015751 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 5...
    2014-03-20 22:13:46.015854 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:46.515744 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:46.515835 [2623/0x9ffa130] flom_client_discover_udp: sending discovery message number 6...
    2014-03-20 22:13:46.515933 [2623/0x9ffa130] flom_client_discover_udp: waiting reply...
    2014-03-20 22:13:47.015690 [2623/0x9ffa130] flom_client_discover_udp: no answer from UDP/IP multicast discovery
    2014-03-20 22:13:47.015843 [2623/0x9ffa130] flom_client_discover_udp/excp=5/ret_cod=4/errno=11 ('Resource temporarily unavailable')
    2014-03-20 22:13:47.015906 [2623/0x9ffa130] flom_client_connect: connection failed, a new daemon can not be started because daemon lifespan is 0
    2014-03-20 22:13:47.015930 [2623/0x9ffa130] flom_client_connect/excp=4/ret_cod=-104/errno=11
    flom_client_connect: ret_cod=-104 (ERROR: 'connect' function returned an error condition)
    tiian@mojan:~/src/flom$

### DiscoveryTimeout
**Command line option**: "-I", "\-\-discovery-timeout"   
**Meaning**: how long a FLoM command (client) waits for an answer UDP datagram after it sent a request UDP multicast datagram.   
Shorter timeouts can be used in presence of low latency networks while longer timeouts should be used when FLoM is used with high latency networks. Timeout is expressed in milliseconds.

### DiscoveryTTL
**Command line option**: "\-\-discovery-ttl"   
**Meaning**: TTL associated to datagrams sent during auto-discovery UDP multicast query; this option can be useful if your datagrams has to cross some routers (a typical usage is inside a WAN scenario).

## UDP/IP Multicast and firewall related issues
Some Linux distributions set a default firewall configuration that stops IP 4 multicast traffic and does not allow FLoM network dynamic modes.    
You have to properly configure internal Linux firewall to allow multicast if you want to use it.   
Sometimes firewalling is not needed at all and you can disable it completely; here are some hints:   
[CentOS7 / CentOS8 / OpenLogic 7 / Red Hat 7](http://www.server-world.info/en/note?os=CentOS_7&p=initial_conf&f=2)    
CentOS 6.6, if **systemctl** command is not available you can use these ones:   

    service ipchains stop
    service iptables stop
    chkconfig ipchains off
    chkconfig iptables off

**Your attention please:** I'm **NOT** telling you that you have to disable your firewalling policy, I'm only telling you have to fix them if you want to use UDP/IP multicast.
