---
layout: page
title: V. Interfaces and Services
theme: dark-green
sidebar: docs_sidebar
permalink: docs/design/design-6.html
nav: design/design-6.html

prev: design-5.html
---

We shall now go into detail about interfaces and services in FreeType.

*Interfaces* in FreeType are analogous to those found in object-oriented
programming. They can be thought of as internal public APIs, and are
essentially tables of function pointers.

Interfaces only describe the form and functionality, but the actual
function body may be implemented elsewhere. The module that is
implementing the interface will then pass the required function pointers
to the table. This gives modularity and easy extendability.

There are two main kinds of interfaces: *module* interfaces, and
*services*.

Module interfaces are defined for each module. For example, every font
driver provides its own set of procedures for use in the base layer,
which are registered in an `FT_Driver`. This way, very different font
drivers can be used in the same way in the base layer.

Services are cross-module interfaces. These provide functionality needed
in several font drivers.

Services are created when code from one module needs to be used in
another. Rather than include files from another module, a service is
created instead. Now, the other module just needs to include the header
defining the interface.

Helper modules are an extreme example of this; all their public
functionality is made for use in other font drivers and hence are in a
single service.

### In-depth guide: Creating a service

This section will teach you how to write your own service.

1.  Make the service interface header.

    We will be calling our service demo for demonstration purposes.
    First, create the header file, which goes in
    `include/freetype/internal/services`, and add the boilerplate.

    ```c
    #include FT_INTERNAL_SERVICE_H
    FT_BEGIN_HEADER
    #define FT_SERVICE_ID_DEMO  "demo"

    /* ...  */

    FT_END_HEADER
    ```

    This line in particular is required to register the new service
    later on.

    ```c
    #define service-id service-tag
    ```

2.  We will have identified some functions that are needed in another
    module. Extract the function signatures of these and place them in
    the header.

    ```
    [typedef return-type
    (*type-name)(function-signature);]+
    ```

    Example:

    ```c
    typedef FT_Error
    (*SampleDoSomethingFunc)( int  foo );
    typedef void
    (*SampleDoAnotherFunc)( int    foo,
                        float  bar );
    ```

3.  Define the service interface.

    Use the `FT_DEFINE_SERVICE` macro to do this.

    ```c
    FT_DEFINE_SERVICE( service-name )
    {
    [type-name  interface-entry;]+
    }
    ```

    Example:

    ```c
    FT_DEFINE_SERVICE( Demo )
    {
    SampleDoSomethingFunc  doSomething;
    SampleDoAnotherFunc    doAnother;
    };
    ```

    Here is the definition of the above macro (in `ftserv.h`).
    
    ```c
    #define FT_DEFINE_SERVICE( name )            \
    typedef struct FT_Service_ ## name ## Rec_ \
    FT_Service_ ## name ## Rec ;             \
    typedef struct FT_Service_ ## name ## Rec_ \
    const * FT_Service_ ## name ;            \
    struct FT_Service_ ## name ## Rec_
    ```

    This defines a new struct called `FT_Service_DemoRec`, along with a
    handle type for the struct, `FT_Service_Demo`.

4.  Register (and/or implement) the functions for the newly created
    interface.

    In the module implementing the interface, add a definition like the
    following

    ```c
    static const service-record  table-name =
    {
    function-name
    [,function-name2 [,...]]
    }
    ```

    which corresponds to the interface defined in step 3. Example:

    ```c
    static const FT_Service_DemoRec  demo_service_rec =
    {
    function1,
    function2
    };
    ```

    This initializes the function pointers table. In this example,
    `function1` has the function signature of `SampleDoSomethingFunc`
    and implements the `doSomething` functionality, and so on.

5.  Register the new service.

    Next, add this code to define the service list in this module, or
    append the new service to an existing list.

    ```c
    static const FT_ServiceDescRec service-list-name[] =
    {
    { service-id, &table-name },
    [{ service-id2, &table-name2 }, [...]]
    { NULL, NULL }
    }
    ```

    Example:

    ```c
    static const FT_ServiceDescRec demo_services[] =
    {
    { FT_SERVICE_ID_DEMO, &demo_service_rec },
    { NULL, NULL }
    };
    ```

    *service-id* is what we `#define`\'d in the service header file in
    step 1. Note that the `{NULL, NULL}` sentinel value at the end is
    required.

    Now we need a way for other modules to find this service. First, we
    need to implement the `get_interface` function in `FT_Module_Class`.
    Here is a minimal example that does not do any validation.

    ```c
    FT_CALLBACK_DEF( FT_Module_Interface )
    get-interface-name( FT_Module    module,
                    const char*  module_interface )
    {
    return ft_service_list_lookup( service-list-name,
                                module_interface );
    }
    ```

    Then, pass it into the `FT_DEFINE_MODULE` macro for this module.

    ```
    (FT_Module_Requester) get-interface-name
    ```

    The last step is optional but recommended, which is to register the
    new service header in `ftserv.h`.

    ```c
    #define FT_SERVICE_DEMO_H  <freetype/internal/services/svdemo.h>
    ```

6.  Use the new service.

    Now, in the file that wants to use the service, add the following
    code to get the service.

    ```c
    service-record-handle  service;


    FT_FACE_FIND_GLOBAL_SERVICE( face,
                            service,
                            service-id-tail );
    ```

    `face` should be of type `FT_Face`, which is usually the current
    face instance being used in the driver, and FreeType tries to find
    the service in the driver of this face first, before searching all
    other modules.

    *service-id-tail* is the part of *service-id* following
    `FT_SERVICE_ID_`.

    Now to call some function in the service.

    ```c
    service->interface-entry( params );
    ```

    Example:

    ```c
    FT_Service_Demo  demo;
    FT_Error         error;


    FT_FACE_FIND_GLOBAL_SERVICE( face, demo, DEMO );

    error = demo->doSomething(0);
    ```
