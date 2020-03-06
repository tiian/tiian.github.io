# The libflom programming kompass for Java developers

FLoM API is very easy to understand and use; these are the basic concepts:

* *flom.jar* is the archive you must include to build your application
* *FlomHandle* is the class you must use to communicate with FLoM (you must treat this type of objects as opaque objects: for every useful property there's a getter/setter method and you don't have to mind what's the meaning of the object content)
* [http://www.tiian.org/flom/API/java/](http://www.tiian.org/flom/API/java/) is the home for Javadoc style documentation

FLoM Java API is tested with these Java versions: 6, 7, 8; the JVM implementation used for development and test is [OpenJDK](http://openjdk.java.net/)

## Basic example

[Basic.java](https://github.com/tiian/flom/blob/master/doc/examples/java/Basic.java.in) example can be considered the *hello world FLoM equivalent* for *static programming style*:

1. allocate a *FlomHandle* object on the stack declaring it
2. synchronize your program using method *lock()*
3. do everything your program must perform inside the synchronization window
5. desynchronize your program using method *unlock()*

The source code is installed in *doc/examples/java* directory (typically */usr/local/share/doc/flom/examples/java*); look inside the source code to retrieve build and execution information.

## Advanced example
The Basic example do not explain as you can specify the name of the logical resource, the quantity of a numerical resource and the other parameters you typically use with **flom** command line.    
[Advanced.java](https://github.com/tiian/flom/blob/master/doc/examples/java/Advanced.java.in) example shows how some properties can be set before entering in the synchronization phase.    
The complete list of the available setter/getter methods can be retrieved from:

* [FLoM classes](http://www.tiian.org/flom/API/java/) Java API page
* [FlomHandle.java](https://github.com/tiian/flom/blob/master/src/java/org/tiian/flom/FlomHandle.java) source file

# Transactional example
Some resources, for example unique sequences, can be declared as *transactional*.    
A transactional resource has an associated state and it needs an explicit *unlock* to commit the state.    
If a transactional resource is not committed due to a program crash, the state will be rolled back.   
Method **unlockRollback** has been introduced to force a state rollback during the **unlock** phase, even if a program does not crash.   
An example is available in this source code: [Transactional.java](https://github.com/tiian/flom/blob/master/doc/examples/java/Transactional.java.in).

