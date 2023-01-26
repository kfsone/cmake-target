Demonstration of using CMake interface targets as metadata collections,
rather than having to learn all the variables associated with a target.

The Old
-------

Early CMake libraries would define a whole slew of variables that you had to be
aware of an integrate into your own cmake files:

```
### OLD: DO NOT USE
find_package (Python3)
...

add_executable(MyExe main.cpp)
# Make sure WE set the right compiler flags...
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PYTHON_COMPILER_FLAGS}")
# Make sure WE tell the compiler where python's includes are...
include_directories(${PYTHON_INCLUDE_DIRS})
...
```

But modern CMake emphasizes associating variables (aka properties) and all that
compile-time information _with_ your target at the definition point, so that
consumers can simply bring it all in when they _link_ to your target.

The point of confusion for everyone is that this makes the terms "library"
and "link" overloaded.

Worse, if you are new to CMake, you're expected to discover this thru reading
about the 'VISIBILITY' properties of libraries, which I'm sure was the first
thing you looked up in the manpages, right?


```
# # Old #1: Set a global variable with my include directories, consumers need
# # to reference it.

#provider
add_library(somelib ...)
set (SOMELIBS_LIBS         somelib pthread ...)
set (SOMELIBS_CFLAGS       $SOMELIBS_CFLAGS -DUSE_SOMELIB=1 -Werror)   # because that won't hurt will it?
set (SOMELIBS_INCLUDE_DIRS ${SOMELIBS_INCLUDE_DIRS} /somewhere/over/the/rainbow)

#consumer
set (CMAKE_CFLAGS ${CMAKE_CFLAGS} ${SOMELIBS_CFLAGS} ...)
include_directories (... ${SOMELIBS_INCLUDE_DIRS}

add_executable (my_prog main.cpp)

target_link_libraries (my_prog ${SOMELIB_LIBS})
```

```
# # Old #2: Just force everything to build the way your library wants

# provider
add_library (somelib ...)
include_directories (/somewhere/over/the/rainbow)
set (CMAKE_CFLAGS ${CMAKE_CFLAGS} -DUSE_SOMELIB=1 -Werror)  # world of hurt
set (SOMELIBS_LIBS somelib pthread ...)  # still have to use variables too

#consumer
add_executable (my_prog main.cpp)
target_link_libraries (my_prog ${SOMELIB_LIBS})
```

```
# # Modern

# provider
add_library (somelib ...)
target_link_libraries (somelib PUBLIC pthread ...)
target_compile_options (somelib PRIVATE -Werror)  # you don't have to -Werror with me
target_compile_definitions (somelib PUBLIC -DUSE_SOMELIB=1)  # you'll need this to use me
target_include_directories (my_compile_flags /somewhere/over/the/rainbow)


# consume
add_executable (my_prog main.cpp)
target_link_libraries (my_prog somelib)
```


Doing the experiment
--------------------

My standard recommended build method is:

```
> cmake -G Ninja -DCMAKE_VERBOSE_MAKEFILE=ON -S . -B build
> cmake --build ./build
```

Compile and build. The program doesn't do anything, but it won't build & link
if it didn't have access to Python correctly.
