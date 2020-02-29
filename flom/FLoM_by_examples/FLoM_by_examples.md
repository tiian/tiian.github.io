FLoM by examples
===

Have you already installed FLoM? (See [installation](../Installation.md) otherwise...)

Now you are ready to try some use cases related to FLoM local usage:

* user command/script synchronization: [Use Case 1](Use_Case_1.md)
* user multiple command/script synchronization: [Use Case 2](Use_Case_2.md)
* don't wait if the resource is busy: [Use Case 3](Use_Case_3.md)
* use a timeout if the resource is busy: [Use Case 4](Use_Case_4.md)
* non exclusive locking: [Use Case 5]
* multi-user command/script synchronization: [Use Case 6]

FLoM can be used also in a distributed environment: the same synchronization features are available inside a single system and through a network (LAN, MAN, WAN and Internet).
You may deep into some [distributed features] of FLoM or you can jump directly to use cases:

* network mode, unicast only: [Use Case 7] and [IPv6 Use Case 7]
* network mode with autodiscovery, unicast and multicast: [Use Case 8] and [IPv6 Use Case 8]
* dynamic network mode, unicast and multicast: [Use Case 9] and [IPv6 Use Case 9]

FLoM provides *numeric resources* useful to limit the number of concurrent processes at a desired value:

* numeric resource, basic usage: [Use Case 10]
* numeric resource, one writer and exactly N readers: [Use Case 11]

FLoM provides *resource sets* useful to exclusively access to a set of named resources:

* resource sets, basic usage: [Use Case 12]
* resource sets, passing arguments: [Use Case 13]

FLoM provides *hierarchical resources* useful to model filesystem objects (files and directorories) and/or scenarios where you have to deal with an object hierarchy:

* hierarchical resources, simple file usage: [Use Case 14]
* hierarchical resources, hierarchical locks: [Use Case 15]

FLoM provides "future events" useful to model synchronization with task that can start in the future:

* future events, simple usage: [Use Case 16]
* future events, long life resources: [Use Case 17]

FLoM provides *sequence resources* useful to synchronize and to get a unique sequence number with the same atomic operation:

* sequence resource: [Use Case 18]

FLoM provides *timestamp resources* useful to synchronize and to get a distinct timestamp with the same atomic operation:

* timestamp resource: [Use Case 19]
