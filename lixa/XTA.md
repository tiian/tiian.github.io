# XTA - XA Transaction API

XTA is a new API that provides two phase commit feature to applications.
In comparison with other existent approaches, XTA has been designed to work in
a *serverless* environment with a FaaS (Function as a Service) model.
Moreover, the two phase commit feature is supplied to the applications with
a TXaaS (Transaction as a Service) model.
Here are the most valuable characteristics:

* Serverless
* Distributed Applications
* Distributed Resources
* Transaction as a Service
* Polyglot
* XA Resource Managers
* non XA Resource Managers

### Serverless

Applications do **not** have to use a specific *application server* to use XTA.
XTA is just a library that must be linked by applications.

### Distributed Applications

Any number of different Applications, distributed in different systems, can
participate in the same global XA transaction. XTA requires only TCP/IP
connectivity.

### Distributes Resources

Any number of different Resources like SQL databases, NoSQL databases,
messaging brokers, etc... can participate in the same global XA transaction.

### Transaction as a Service

XTA uses the LIXA state server to persist transaction state and to synchronize
different applications. LIXA state server is accessed by XTA using an XML
message oriented, protocol based on standard TCP/IP connection.

### Polyglot

Application developed with different programming languages can participate in
the same global XA transacation.
Currently available languages are: C, C++ and Python.

### XA Resource Managers

XTA natively recognizes XA standard Resource Managers.

### non XA Resource Managers

XTA provides an interface that can be implemented by any Resource Manager that
can support two phase commit indipendently from XA standard. 

## XTA documentation

XTA documentation is available inside the
[LIXA manual](/manuals/html/index.html)
