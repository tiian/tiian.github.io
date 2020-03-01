# FLOM can operate in 4 different ways:

* **local only**: the lock manager operates inside a single running system, this is not a distributed feature at all
* **network**: the lock manager operates inside an IP network that may contain any number of running systems. At least one node must be chosen to host *flom daemon*, other nodes must point to a *flom daemon node*. This option gives you a **distributed lock manager** you can use it to synchronize activities happening inside different systems
* **network with autodiscovery**: this mode extends *network* mode adding the support for *flom daemon autodiscovery feature*. This option adds you a lot of flexibility: you can change *flom daemon* host without reconfiguring other systems: *flom daemon* is discovered using UDP/IP multicast protocol
* **dynamic network**: this mode gives you a *zero conf distributed lock manager* because the *flom daemon* will be automatically activated when necessary and automatically discovered by other running systems.

## Dynamic network mode warning

It might seem that *dynamic network* is a sort of *catch all* solution: you could discover how easy and powerful it is just reading the related [Use Case 9](FLoM_by_examples/Use_Case_9.md).
Unfortunately it's an ***unsafe operation mode***: you should use it only if an error in synchronization does **not** corrupt your data and does not allows your computation to produce inconsistent results.

** Repeat: dynamic network mode can corrupt your data and/or break your algorithms. **

But if you are using the *flom distributed lock manager* to synchronize processes that can coexist without any side effect (a typical scenario is workload balancing issues), *dynamic network mode* can give you unbeatable flexibility and ease of use.

### Why dynamic network mode is dangerous

Referring to [CAP Theorem](http://en.wikipedia.org/wiki/CAP_theorem) the biggest issue correlated to *dynamic network mode* is related to *network partitions*: if, for any reason, your network splits from an UDP/IP multicast point of view (some systems are not able to communicate with others using UDP/IP multicast or UDP/IP multicast are slowed down), you will have extra running *flom daemon instances* and **your processes will synchronize in different unpredictable clusters**. The depicted scenario is *inconvenient* and now you do know it!

*FLOM* does **not** try to implement some complex algorithm like *totally ordered multicast* to manage this type of issues: it simply perform a request/reply with timeout: if reply message don't arrive before timeout expiration, the requester will activate a new *flom daemon*.

## Local and network configuration modes

There are basically 3 different options that influence operation mode.
This first table resume the command line option and configuration file keywords necessary to specify local and network modes:

Config type|Unicast (TCP/IP) address|Multicast (UDP/IP) address|Lifespan
-|-|-|-|-
**command line option (short form)**|-a|-A|-d
**command line option (long form)**|--unicast-address|--multicast-address|--daemon-lifespan
**config file keyword**|UnicastAddress|MulticastAddress|Lifespan

This second table explains the behavior obtained changing configuration options:

Unicast address|Multicast address|Lifespan|Behavior
-|-|-|-
null|null|< 0|local mode (no network): if a *flom daemon* is not locally available a new one will be started and it will not spontaneously end
null|null|= 0|local mode (no network): if a *flom daemon* is not locally available a new one will **not** be started and the command will fail
null|null|> 0|local mode (no network): if a *flom daemon* is not locally available a new one will be started and it will end after the inactivity time exceeds the *Lifespan* specified value
IP address/name|null|< 0|network mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will be started locally trying to bind it to the specified *IP address/name*; if started, the daemon would not spontaneously end. **Note:** *IP address/name* must be a valid address/name for at least one of the network interfaces locally available
IP address/name|null|= 0|network mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will **not** be started locally and the command will fail
IP address/name|null|> 0|network mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will be started locally trying to bind it to the specified *IP address/name*; if started, the daemon would spontaneously end after the inactivity time exceeds the *Lifespan* specified value. **Note:** *IP address/name* must be a valid address/name for at least one of the network interfaces locally available
IP address/name|IP address/name|< 0|network with autodiscovery mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will be started locally trying to bind it to the specified *IP address/name*; if started, the daemon would not spontaneously end. If activated, the daemon would answer to UDP/IP multicast query: *autodiscovery feature*. **Note:** *IP address/name* must be a valid address/name for at least one of the network interfaces locally available
IP address/name|IP address/name|= 0|network with autodiscovery mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will **not** be started locally and the command will fail. **Note:** multicast UDP/IP address will be ignored because unicast TCP/IP address is specified
IP address/name|IP address/name|> 0|network with autodiscovery mode: if a *flom daemon* is not available at the specified *IP address/name* a new one will be started locally trying to bind it to the specified *IP address/name*; if started, the daemon would spontaneously end after the inactivity time exceeds the *Lifespan* specified value. If activated, the daemon would answer to UDP/IP multicast query: *autodiscovery feature*. **Note:** *IP address/name* must be a valid address/name for at least one of the network interfaces locally available
null|IP address/name|< 0|dynamic network mode: if a *flom daemon* does not reply to UDP/IP multicast query at the specified *IP address/name* a new one will be started locally trying to bind all the available interfaces (INADDR_ANY); if started, the daemon would not spontaneously end. If activated, the daemon would answer to UDP/IP multicast query: *autodiscovery feature*. 
null|IP address/name|= 0|dynamic network mode: if a *flom daemon* does not reply to UDP/IP multicast query at the specified *IP address/name* a new one will **not** be started locally and the command will fail
null|IP address/name|> 0|dynamic network mode: if a *flom daemon* does not reply to UDP/IP multicast query at the specified *IP address/name* a new one will be started locally trying to bind all the available interfaces (INADDR_ANY); if started, the daemon would spontaneously end after the inactivity time exceeds the *Lifespan* specified value. If activated, the daemon would answer to UDP/IP multicast query: *autodiscovery feature*

### Additional details
When operating in *local mode*, configuration option "-s", "\-\-socket-name", "SocketName" can be used to change the default behavior; default behavior uses a per user socket name and, as a consequence, a per user *flom daemon*. See [Use Case 6] for additional information.

The following table resumes the configuration options necessary to customize the ports used for TCP/IP unicast and UDP/IP multicast communications:

Config type|Unicast (TCP/IP) port|Multicast (UDP/IP) port
-|-|-|-
**command line option (short form)**|-p|-P
**command line option (long form)**|--unicast-port|--multicast-port
**config file keyword**|UnicastPort|MulticastPort

#### Note related to ports
Configuration options related to ports are meaningful only if the correspondent configuration options related to addresses are set.

*flom man page* is the official location to discover the default port for your *flom* version.

### UDP/IP multicast and firewalls
Please **pay attention** some Linux distros configure a default firewall that **silently drop** multicast datagrams: disable your firewall or configure it to **not** drop multicast datagrams.
**openSUSE** 12.2 is such a distribution.

