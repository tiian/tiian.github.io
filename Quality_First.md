# Quality First

Projects developed inside www.tiian.org strictly follow some principles and rules to provide a pre-defined quality level. The principles and the rules are below reported to guarantee 100% transparency about what you get using the software.

## Documentation is a Must

Every documentation error is managed with the same process of bug fixing: as soon as it has been detected, an activity to fix it is put in the pipeline.

## Licenses: Clear and Transparent

GNU GPL and LGPL licenses are used: no need to search something different, the same licenses used by the Linux kernel and many Linux core components.
Every source file specifies the applicable license.

## Observability Features

### Logging

At run-time several logging messages, with different severities, are sent to the default system logger; logging is the core capability to catch anomalous condition in production environments.

### Tracing

The source code contains a huge number of trace messages that can be activated/deactivated selectively; tracing is the core capability to understand what's going wrong when something is not working as expected.

## Usability

Every feature is double and triple checked at design time to look for the easier to use approach. Some complexity can not be avoided, but the effort to reduce useless complexity is constantly applied.

## Source Code Rules

### Comments

The source code contains abundant comments and all the relevant items (functions, structs, classes, methods, etc...) are self documented using standard tools like Doxygen and Javadoc.

### Security

Source code is automatically scanned with CodeQL provided by github.

### Style

The source code is continuously reviewed to guarantee uniform style: it's not a matter of embellishment, but a matter of reading and understanding it.

