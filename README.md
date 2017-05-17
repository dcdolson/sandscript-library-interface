sandscript-library-interface
============================

This provides the header files for interfacing Sandvine's SandScript Policy
Engine to arbitrary external shared libraries.

# The Public Interfaces

The SandScript Policy Engine will `dlopen()` the library and seek these
symbols:
 - `GetManifest`
 - `GetEventManifest`

These are both functions. You may implement one or both of these.
 
GetManifest must be a symbol of type `psl_GetManifest`, specifically a function
that returns the manifest of all functions in the library. You will need to
implement this if you want to extend SandScript with **functions** a SandScript
author can call.

GetEventManifest must be a symbol of type `psl_GetEventManifest`, a function
that returns the manifest of all of the events in the library.  You will need
to implement this if you want to extend SandScript to **raise events** that can be
handled in SandScript.

The library developer's primary role is to fill in the manifests with all of
the information to describe the functions and events provided by the shared
library. Please refer to the header files for more information:
 - `sharedLibTypes.h` -- common data types
 - `sharedLibManifest.h` -- for declaring functions
 - `sharedLibEvents.h` -- for declaring events

# Naming Functions and Fields

When you extend the Policy Engine, you want to avoid overlap with any other
libraries or built-in subsystems. For this reason, we recommend the following
naming conventions:
 - choose a name for the subsystem (E.g., "Foo")
 - all functions begin with the subsystem name (E.g., "Foo.Func()")
 - choose a name for each event type (E.g., "Foo.Event1", "Foo.Event2")
 - all fields begin with the event prefix the belong to (E.g.,
   "Foo.Event1.Field1")

# Doxygen

The shared library interface is documented using doxygen, and a doxygen
configuration file is provided. The following command will create html
documentation in a sub-folder `documentation`.

    doxygen doxygen.cfg

# Compiling

You are building a shared library, so (assuming you are using gcc) it must be
linked with `-shared` and `-fPIC`. We also recommend turning on all warnings

    gcc -fPIC -Wall -shared -o my_shared_lib.so source1.c source2.c ...

Deploy the shared library in `/usr/local/sandvine/loadable/` to be picked up
the next time the policy engine is started.

# Sanity-Checking

To 'test' your library, there is a simple tool that will open your shared
library and read the manifests.

In dump_util/
You can compile `dump-shared-library-contents.c` as:

    make

and then run it as:

    ./dump-shared-library-contents <mylib>

and it will dump out the data structures as the policy engine will see them.

This tools walks the manifests but does not actually call the functions you
have defined.

Note: once you have compiled the tool, you can test it on different versions of
your library without re-building it.

