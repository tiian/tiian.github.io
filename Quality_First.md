# Quality First

Projects developed inside www.tiian.org strictly follow some principles and rules to provide a pre-defined quality level. The principles and the rules are below reported to guarantee 100% transparency about what you get using these softwares.

## Clear and Transparent Licenses

GNU GPL and LGPL licenses are used: no need to search something different, the same licenses used by Linux kernel and many Linux core components.
Every source file specifies the applicable license.

## Observability Features

### Logging

At run-time several logging messages, with different severities are sent to the default system logger; logging is the core capability to catch anomalous condition in production environment.

### Tracing

The source code contains tons of trace messages that can be activated/deactivated selectively; tracing is the core capability to understand what's going wrong when something is not working as expected.

## Source Code Rules

### Comments

The source code contains abundant comments and all the relevant items (functions, structs, classes, methods, etc...) are self documented using standard tools like Doxygen and Javadoc.

### Style

The source code is continuously reviewed to guarantee uniform style: it's not a matter of embellishment, but a matter of reading and understanding it.

