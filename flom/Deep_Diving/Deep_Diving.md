# FLoM deep diving

[Integrating](Integrating.md) FLoM with a scheduler like Jenkins require the usage of option "\-\-ignore-signal".

To have a look about what's happening inside a FLoM daemon, like for example how many clients are waiting for a resource, you can turn on the [observability](Observability.md) feature available starting with version 1.7.0

Specific [network options](Network_Options.md) can be useful to change default behavior and use FLoM in non trivial situations.

[IPv6 multicast](IPv6_Multicast.md) feature is supported by a specific debugging tool to fix your system configuration.

SSL/TLS Security is supported by a specific debugging tool to help in fixing your system configuration:

* [Channel Encryption Debug](Channel_Encryption_Debug.md)
* [Mutual Authentication Debug](Mutual_Authentication_Debug.md)

