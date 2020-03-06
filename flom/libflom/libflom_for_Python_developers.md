# The libflom programming kompass for Python developers

FLoM API is very easy to understand and use; these are the basic concepts:

* *flom* is the package you must include
* *flom_handle_t* is the data type you must use to communicate with FLoM (you must treat this type of object as an opaque object: for every useful property there's a getter/setter method and you don't have to mind what's the meaning of the object content)

## Basic programming example

[basic.py](https://github.com/tiian/flom/blob/master/doc/examples/python/basic.py) example can be considered the *hello world FLoM equivalent* for *Python programming*:

1. allocate a *flom_handle_t* object
2. initialize the object using method (function) *flom_handle_init*
3. synchronize your program using method (function) *flom_handle_lock*
4. do everything your program must perform inside the synchronization window
5. desynchronize your program using method (function) *flom_handle_unlock*
6. clean-up the object using method (function) *flom_handle_clean*

**Note:** you must clean-up every initialized *flom_handle_t* object; if you don't clean-up every object you will generate memory leaks.

## Advanced programming example
Basic example does not explain as you can specify the name of the logical resource, the quantity of a numerical resource and the other parameters you typically use with **flom** command line.    
[advanced.py](https://github.com/tiian/flom/blob/master/doc/examples/python/advanced.py) example shows how some properties can be set before entering in the synchronization phase.    
The complete list of the available setter/getter methods can be retrieved from:

* [functions](http://www.tiian.org/flom/API/C/globals_func.html) C API page

**Note:** Python libflom wrapper exposes the same functions provided by the C basic library.

## Transactional example
Some resources, for example unique sequences, can be declared as *transactional*.    
A transactional resource has an associated state and it needs an explicit *unlock* to commit the state.    
If a transactional resource is not committed due to a program crash, the state will be rolled back.   
Function **flom_handle_unlock_rollback** has been introduced to force a state rollback during the **unlock** phase, even if a program does not crash.   
An example is available in this source code: [transactional.py](https://github.com/tiian/flom/blob/master/doc/examples/python/transactional.py).

