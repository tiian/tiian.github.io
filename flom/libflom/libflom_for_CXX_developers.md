# The libflom programming kompass for C++ developers

FLoM API is very easy to understand and use; these are the basic concepts:

* *flom.hh* is the header file you must include
* *FlomHandle* is the class you must use to communicate with FLoM (you must treat this type of objects as opaque objects: for every useful property there's a getter/setter method and you don't have to mind what's the meaning of the object content)
* you can use a "static programming style", allocating *FlomHandle* objects on the stack or you can use a "dynamic programming style", allocating *FlomHandle* objects in the heap: that's your choice and FLoM API does not force you to use a specific style

## Static programming style example

[BasicStatic.cc](https://github.com/tiian/flom/blob/master/doc/examples/BasicStatic.cc) example can be considered the *hello world FLoM equivalent* for *static programming style*:

1. allocate a *FlomHandle* object on the stack declaring it
2. synchronize your program using method *lock()*
3. do everything your program must perform inside the synchronization window
5. deserialize your program using method *unlock()*

**Note:** C++ clean-ups every statically initialized *FlomHandle* object for you: you don't have to mind about memory leaks.

## Dynamic programming style example

[BasicDynamic.cc](https://github.com/tiian/flom/blob/master/doc/examples/BasicDynamic.cc) example can be considered the *hello world FLoM equivalent* for *dynamic programming style*:
1. declare a *FlomHandle* object pointer
2. create a new object using *new*
3. synchronize your program using method *lock()*
4. do everything your program must perform inside the synchronization window
5. desynchronize your program using method *unlock()*
6. delete the object using *delete*

**Note:** you must delete every created *FlomHandle* object; if you don't delete every object you will generate memory leaks.

# Advanced examples
Basic examples do not explain as you can specify the name of the logical resource, the quantity of a numerical resource and the other parameters you typically use with **flom** command line.    
[AdvancedStatic.cc](https://github.com/tiian/flom/blob/master/doc/examples/AdvancedStatic.cc) and [AdvancedDynamic.cc](https://github.com/tiian/flom/blob/master/doc/examples/AdvancedDynamic.cc) examples show how some properties can be set before entering in the synchronization phase.    
The complete list of the available setter/getter methods can be retrieved from:

* [FlomHandle class](http://flom.sourceforge.net/html/C++/) API page
* [FlomHandle.hh](https://github.com/tiian/flom/blob/master/src/FlomHandle.hh) header file

# Transactional example
Some resources, for example unique sequences, can be declared as *transactional*.    
A transactional resource has an associated state and it needs an explicit *unlock* to commit the state.    
If a transactional resource is not committed due to a program crash, the state will be rolled back.   
Method **unlockRollback** has been introduced to force a state rollback during the **unlock** phase, even if a program does not crash.   
An example is available in this source code: [Transactional.cc](https://github.com/tiian/flom/blob/master/doc/examples/Transactional.cc).

