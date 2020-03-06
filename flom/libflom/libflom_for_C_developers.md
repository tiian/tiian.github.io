# The libflom programming kompass for C developers

FLoM API is very easy to understand and use; these are the basic concepts:

* *flom.h* is the header file you must include
* *flom_handle_t* is the data type you must use to communicate with FLoM (you must treat this type of objects as opaque objects: for every useful property there's a getter/setter method and you don't have to mind what's the meaning of the object content)
* you can use a "static programming style", allocating *flom_handle_t* objects on the stack or you can use a "dynamic programming style", allocating *flom_handle_t* objects in the heap: that's your choice and FLoM API does not force you to use a specific style

## Static programming style example

[basic_static.c](https://github.com/tiian/flom/blob/master/doc/examples/basic_static.c) example can be considered the *hello world FLoM equivalent* for *static programming style*:

1. allocate a *flom_handle_t* object on the stack declaring it
2. initialize the object using method (function) *flom_handle_init*
3. synchronize your program using method (function) *flom_handle_lock*
4. do everything your program must perform inside the synchronization window
5. desynchronize your program using method (function) *flom_handle_unlock*
6. clean-up the object using method (function) *flom_handle_clean*

**Note:** you must clean-up every initialized *flom_handle_t* object; if you don't clean-up every object you will generate memory leaks.

## Dynamic programming style example

[basic_dynamic.c](https://github.com/tiian/flom/blob/master/doc/examples/basic_dynamic.c) example can be considered the *hello world FLoM equivalent* for *dynamic programming style*:

1. declare a *flom_handle_t* object pointer
2. create a new object using method (function) *flom_handle_new*
3. synchronize your program using method (function) *flom_handle_lock*
4. do everything your program must perform inside the synchronization window
5. desynchronize your program using method (function) *flom_handle_unlock*
6. delete the object using method (function) *flom_handle_delete*

**Note:** you must delete every created *flom_handle_t* object; if you don't delete every object you will generate memory leaks.

### Static and dynamic programming styles

**Note:** the availability of two different programming styles does **not** mean you can mix them!    
If you defined a static *flom_handle_t* object, you could not apply *flom_handle_delete* method against it.   
If you created a dynamic *flom_handle_t* object, you could not apply *flom_handle_init* and *flom_handle_clean* methods against it.   
Don't mix the styles, that pattern is **not** supported.

# Advanced examples
Basic examples do not explain as you can specify the name of the logical resource, the quantity of a numerical resource and the other parameters you typically use with **flom** command line.    
[advanced_static.c](https://github.com/tiian/flom/blob/master/doc/examples/advanced_static.c) and [advanced_dynamic.c](https://github.com/tiian/flom/blob/master/doc/examples/advanced_dynamic.c) examples show how some properties can be set before entering in the synchronization phase.    
The complete list of the available setter/getter methods can be retrieved from:

* [functions](http://www.tiian.org/flom/API/C/globals_func.html) C API page
* [flom_handle.h](https://github.com/tiian/flom/blob/master/src/flom_handle.h) header file

# Transactional example
Some resources, for example unique sequences, can be declared as *transactional*.    
A transactional resource has an associated state and it needs an explicit *unlock* to commit the state.     
If a transactional resource is not committed due to a program crash, the state will be rolled back.    
Function **flom_handle_unlock_rollback** has been introduced to force a state rollback during the **unlock** phase, even if a program does not crash.   
An example is available in this source code: [transactional.c](https://github.com/tiian/flom/blob/master/doc/examples/transactional.c).

