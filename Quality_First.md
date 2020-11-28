# Quality First Approach

Projects developed inside www.tiian.org strictly follow some principles and guidelines to provide a clear quality level.

---

## Quality Principles

### Coherent & Compact Architecture

All the components must cooperate as a whole in a uniform way.

### Easy to Use

Design and implement features that are easy to use for the specific domain: useless complexity does not pay back.

### Full Documentation

Documentation is a core property of software: if it's not documented, there's no way to recognize what *wrong behavior* means.

### Open Minded

Accept to discuss every type of proposal: nothing is *stupid* by itself.

### Transparency

Make every documentation and every discussion public.

### Uncapped Effort

Implementing a feature requires all the necessary effort: no workarounds or shortpath are acceptable.

---

## Quality Guidelines

### Automation

Every phase of the process is automated: build, check, install, distribute; GNU standard toolchain based on *autotools* is used. Automation guarantee reproducibility.

### Compatibility

Software is developed using *the oldest possible* Linux distributions and tested with the newest Linux distributions: this guarantee a wide range of applicability and compatibility. Ubuntu Linux and CentOS are used as reference Linux distributions.

### Documentation is a Must

Every documentation error is managed with the same process of the bug fixing: as soon as it has been detected, an activity to fix it is put in the pipeline.

### Licenses: Clear and Transparent

GNU GPL and LGPL licenses are used: no need to search something different, the same licenses used by the Linux kernel and many Linux core components.
Every source code file specifies the applicable license.

### Observability Features

#### Logging

At run-time several logging messages, with different severities, are sent to the default system logger; logging is the core capability to catch anomalous condition in production environments.

#### Tracing

The source code contains a huge number of trace messages that can be activated/deactivated selectively; tracing is the core capability to understand what's going wrong when something is not working as expected.

### Testing

Every documented use case is tested by one or more dedicated test cases. End-to-end testing is the largely preferred methodology to verify software correctness.

### Usability

Every feature is double and triple checked at design time to look for the easier to use approach. Some complexity can not be avoided, but the effort to reduce useless complexity is constantly applied.

### Source Code Guidelines

#### Comments

The source code contains abundant comments and all the relevant items (functions, structs, classes, methods, etc...) are self documented using standard tools like Doxygen and Javadoc.

#### Safety

Source code is periodically checked with Valgrind to catch memory management and concurrency problems.

#### Security

Source code is automatically scanned with CodeQL provided by github.

#### Style

The source code is continuously reviewed to guarantee uniform style: it's not a matter of embellishment, but a matter of reading and understanding it. The higher the number of coding styles adopted by a software project, the lower the chance to fully understand what every piece of software does.

