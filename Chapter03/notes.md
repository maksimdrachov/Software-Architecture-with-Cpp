## Chapter 3: Functional and Nonfunctional requirements

To replicate our steps to generate documentation from source, you must have CMake, Doxygen, Spinhx, m2r2 and Breathe installed.

### Generating (API) documentation from code

To help others navigate your existing code and use the APIs you provide, a good idea is to provide documentation generated from the comments in your code. There's no better place for such documentation than just right next to the functions and data types it describes, and this helps a lot in keeping them in sync.

How to set it up in a project. Let's assume we keep our sources in `src`, public headers in `include`, and documentation in `doc`. First, let's create a `CMakeLists.txt` file:

```cmake
cmake_minimum_required(VERSION 3.10)

project("Breathe Demo" VERSION 0.0.1 LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
add_subdirectory(src)
add_subdirectory(doc)
```

We've added the `cmake` directory to the path under which CMake looks for its include files.

In the `cmake` subdirectory, we'll create one file, `FindSphinx.cmake`, which we'll use just as the name suggests, since Sphinx doesn't offer one already:

```
find_program(
    SPHINX_EXECUTABLE
    NAMES sphinx-build
    DOC "Path to sphinx-build executable")

# handle REQUIRED and QUIET arguments, set SPHINX_FOUND variable
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(
    Sphinx "Unable to locate sphinx-build executable" SPHINX_EXECUTABLE)
```

Next, let's create our source to generate the documentation. Let's have an `include/breathe_demo/demo.h` file:

```cpp
#pragma once

// the @file annotation is needed for Doxygen to document the free
// functions in this file
/** 
 * @file
 * @brief The main entry points of our demo
 */

/**
 * A unit of performable work 
 */
struct Payload {
    /** 
     * the actual amount of work to perform
     */
    int amount;
};

/** 
 * @brief Performs really important work
 * @param payload the descriptor of work to be performed
 */
void perform_work(struct Payload payload);
```

We prefer to document our types and functions in the header files since they're the interface to our library. The source files are just implementation and they don't add anything new to the interface.

Aside from the preceding files, we also need a simple `CMakeLists.txt` file in src:

```cmake
add_library(BreatheDemo demo.cpp)
target_include_directories(BreatheDemo PUBLIC
        ${PROJECT_SOURCE_DIR}/include)
target_compile_features(BreatheDemo PUBLIC cxx_std_11)
```

Now, let's move to the `doc` folder. First, its `CMakeLists.txt` file, beginning with a check to establish whether Doxygen is available and omitting generation if so:

```
find_package(Doxygen)
if (NOT DOXYGEN_FOUND)
    return()
endif()
```

Next assuming Doxygen was found, we need to set some variables to steer the generation. We want just the XML output for Breathe, so let's set the following variables:

```
set(DOXYGEN_GENERATE_HTML NO)
set(DOXYGEN_GENERATE_XML YES)
```

