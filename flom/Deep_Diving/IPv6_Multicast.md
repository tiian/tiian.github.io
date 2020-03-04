# IPv6 multicast debugger tool
IPv6 multicast presents some characteristics that can be non trivial to deal with.   
Sometimes your systems and/or your network is not configured in the right way to allow the flow of IPv6 UDP multicast packets.   
FLoM uses UDP multicast datagrams for automatic discovery of the daemon location.    
FLoM supplies a specific troubleshooting tool that should help you to understand what's happening in the case of a system/network misconfiguration.

You can activate a listener (server) with this command:

    flom --debug-feature=ipv6.multicast.server -A ff02::1

ff02::1 is just one of the many IPv6 multicast addresses that you can use inside your network.

You can activate a requester (client) with this command:

    flom --debug-feature=ipv6.multicast.server -A ff02::1

adjust ff02::1 to your address as described for the listener (server) part.

If anything is OK, you should obtain something like these messages on the consoles:

    tiian@ubuntu1004:~/flom$ flom --debug-feature=ipv6.multicast.server -A ff02::1
    Arrived datagram is 'HELLO'
    Sent datagram is 'WELCOME'

    tiian@ubuntu1004:~$ flom --debug-feature=ipv6.multicast.client -A ff02::1
    Sent datagram is 'HELLO'
    Arrived datagram is 'WELCOME'

If anything is not OK the tool will freeze waiting a datagram that will never arrive; you can activate a specific trace flag to have a verbose output of every network operation:

This is a sample output for a listener (server):

    tiian@ubuntu1004:~/flom$ export FLOM_TRACE_MASK=0x10000
    tiian@ubuntu1004:~/flom$ flom --debug-feature=ipv6.multicast.server -A ff01::1
    2015-11-18 00:03:53.340151 [937/0x1be1c00] flom_debug_features
    2015-11-18 00:03:53.340180 [937/0x1be1c00] flom_debug_features: name='ipv6.multicast.server'
    2015-11-18 00:03:53.340184 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server
    2015-11-18 00:03:53.340187 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: multicast address='ff01::1', multicast port=28015
    2015-11-18 00:03:53.340222 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server/getaddrinfo() list pointer 0x1bf27f0
    2015-11-18 00:03:53.340225 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: address returned by getaddrinfo():addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff01::1', sin6_scope_id=0
    2015-11-18 00:03:53.340236 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server/getaddrinfo(): [ai_flags=1,ai_family=10,ai_socktype=2,ai_protocol=17,ai_addrlen=28,ai_canonname='{null}'] 
    2015-11-18 00:03:53.340240 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: creating a new socket...
    2015-11-18 00:03:53.340244 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: address returned by getsockname(): addrlen=28; IPv6 address, sin6_port=0, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-18 00:03:53.340250 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: setting SO_REUSEADDR socket property...
    2015-11-18 00:03:53.340253 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: binding to address addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-18 00:03:53.340261 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: address returned by bind() addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-18 00:03:53.340267 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: activating multicast...
    2015-11-18 00:03:53.340305 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: multicast server created, waiting datagram...
    2015-11-18 00:04:28.220235 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: arrived datagram 'HELLO'
    2015-11-18 00:04:28.220245 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: arrived datagram 48 45 4c 4c 4f 
    2015-11-18 00:04:28.220252 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: address returned by recvfrom() addrlen=28; IPv6 address, sin6_port=55573, sin6_flowinfo=0x0, sin6_addr='fe80::5054:ff:feba:34b0', sin6_scope_id=2
    Arrived datagram is 'HELLO'
    2015-11-18 00:04:28.220267 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: sending 'WELCOME' to the client...
    2015-11-18 00:04:28.220275 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server: sent_bytes=7, welcome_msg_len=7
    Sent datagram is 'WELCOME'
    2015-11-18 00:04:28.220310 [937/0x1be1c00] flom_debug_features_ipv6_multicast_server/excp=5/ret_cod=0/errno=2
    2015-11-18 00:04:28.220315 [937/0x1be1c00] flom_debug_features/excp=1/ret_cod=0/errno=2

and the requester (client) related output:

    tiian@ubuntu1004:~$ export FLOM_TRACE_MASK=0x10000
    tiian@ubuntu1004:~$ flom --debug-feature=ipv6.multicast.client -A ff01::1
    2015-11-18 00:04:28.220092 [938/0x1666c00] flom_debug_features
    2015-11-18 00:04:28.220121 [938/0x1666c00] flom_debug_features: name='ipv6.multicast.client'
    2015-11-18 00:04:28.220125 [938/0x1666c00] flom_debug_features_ipv6_multicast_client
    2015-11-18 00:04:28.220127 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: multicast address='ff01::1', multicast port=28015
    2015-11-18 00:04:28.220153 [938/0x1666c00] flom_debug_features_ipv6_multicast_client/getaddrinfo() list pointer 0x16777f0
    2015-11-18 00:04:28.220157 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: address returned by getaddrinfo():addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff01::1', sin6_scope_id=0
    2015-11-18 00:04:28.220168 [938/0x1666c00] flom_debug_features_ipv6_multicast_client/getaddrinfo(): [ai_flags=1,ai_family=10,ai_socktype=2,ai_protocol=17,ai_addrlen=28,ai_canonname='{null}'] 
    2015-11-18 00:04:28.220174 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: setting IPV6_MULTICAST_HOPS to value 1
    2015-11-18 00:04:28.220178 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: setting SO_REUSEADDR to value 1
    2015-11-18 00:04:28.220180 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: binding to address: addrlen=28; IPv6 address, sin6_port=0, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-18 00:04:28.220189 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: address returned by bind() addrlen=28; IPv6 address, sin6_port=0, sin6_flowinfo=0x0, sin6_addr='::', sin6_scope_id=0
    2015-11-18 00:04:28.220195 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: sending 'HELLO' to the server...
    2015-11-18 00:04:28.220197 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: destination address:addrlen=28; IPv6 address, sin6_port=28015, sin6_flowinfo=0x0, sin6_addr='ff01::1', sin6_scope_id=0
    Sent datagram is 'HELLO'
    2015-11-18 00:04:28.220504 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: waiting an answer from the server...
    2015-11-18 00:04:28.220510 [938/0x1666c00] flom_debug_features_ipv6_multicast_client: received 'WELCOME'
    Arrived datagram is 'WELCOME'
    2015-11-18 00:04:28.220517 [938/0x1666c00] flom_debug_features_ipv6_multicast_client/excp=8/ret_cod=0/errno=2
    2015-11-18 00:04:28.220519 [938/0x1666c00] flom_debug_features/excp=1/ret_cod=0/errno=2

# IPv6 multicast well known issues
Some IPv6 multicast well known issues (related to FLoM).

## ff01::1 is node local...
...and will not be sent to other systems! Pay attention, use ff02::1 instead of ff01::1! The table supplied by (wikipedia)[https://en.wikipedia.org/wiki/Multicast_address#IPv6] is useful and easy to understand.

## Check ip6tables if datagrams disappear without "any reason"

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
    [root@centos66-64 tiian]# ip6tables -F
    [root@centos66-64 tiian]# ip6tables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination         
    [root@centos66-64 tiian]#

The first one is the default policy installed by Centos 6.6 and I had to disable it to allow IPv6 multicast datagrams flow.    
This is not a firewalling site at all, but if you want to use firewall protection and FLoM autodiscovery features, you should probably tune your security configuration.

