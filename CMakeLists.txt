cmake_minimum_required(VERSION 3.13)

#########################
# Configuration Options #
#########################

project(rpp LANGUAGES CXX VERSION 0.0.1)

option(RPP_ENABLE_WALL                          OFF)
option(RPP_ENABLE_WERROR                        OFF)
option(RPP_ENABLE_DETAILED_CONCEPTS_DIAGNOSTICS OFF)
option(RPP_ENABLE_TESTING                       OFF)
option(RPP_ENABLE_STATIC_STDLIB                 ON)
option(RPP_ENABLE_EXCEPTIONS                    OFF)
option(RPP_ENABLE_RTTI                          OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# configure custom Standard Library selection
if(DEFINED RPP_STDLIB AND NOT RPP_STDLIB STREQUAL "default")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=${RPP_STDLIB}")
    else()
        message(FATAL_ERROR "RPP_STDLIB must be \"default\" unless using Clang")
    endif()
endif()

# configure custom Standard Library link paths
if(DEFINED RPP_STDLIB_PATH AND EXISTS ${RPP_STDLIB_PATH})
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L ${RPP_STDLIB_PATH}")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -rpath ${RPP_STDLIB_PATH}")
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,-rpath -Wl,${RPP_STDLIB_PATH}")
    else()
        message(SEND_ERROR  "option  : RPP_STDLIB_PATH")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "setting the Standard Library link path on this compiler unsupported")
    endif()
endif()

# configure custom Standard Library include paths
if(DEFINED RPP_STDLIB_INCLUDE AND EXISTS ${RPP_STDLIB_INCLUDE})
    if(
        CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
        CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    )
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem ${RPP_STDLIB_INCLUDE}")
    else()
        message(SEND_ERROR  "option  : RPP_STDLIB_INCLUDE")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "setting the Standard Library link path on this compiler unsupported")
    endif()
endif()

# configure static Standard Library linkage
if(RPP_ENABLE_STATIC_STDLIB)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
    else()
        message(FATAL_ERROR "RPP_ENABLE_STATIC_STDLIB must be \"OFF\" unless using Clang")
    endif()
endif()

# concepts requires C++20
if(NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 20)
endif()
if(CMAKE_CXX_STANDARD LESS 20)
    message(FATAL_ERROR "This project requires C++20")
endif()

# handle enable-all-warnings
if(RPP_ENABLE_WALL)
    if(
        CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
        CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    )
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
    else()
        message(SEND_ERROR  "option  : RPP_ENABLE_WALL")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "enabling all warning on this compiler unsupported")
    endif()
endif()

# handle treat-warnings-as-errors
if(RPP_ENABLE_WERROR)
    if(
        CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
        CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    )
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
    else()
        message(SEND_ERROR  "option  : RPP_ENABLE_WERROR")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "warnings as errors on this compiler unsupported")
    endif()
endif()

# handle detailed-concepts-diagnostics
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if(RPP_ENABLE_DETAILED_CONCEPTS_DIAGNOSTICS)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fconcepts-diagnostics-depth=9")
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    if(RPP_ENABLE_DETAILED_CONCEPTS_DIAGNOSTICS)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ftemplate-backtrace-limit=0")
    endif()
else()
    if(RPP_ENABLE_DETAILED_CONCEPTS_DIAGNOSTICS)
        message(SEND_ERROR  "option  : RPP_ENABLE_DETAILED_CONCEPTS_DIAGNOSTICS")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "concepts diagnostics on this compiler unsupported")
    endif()
endif()

# handle exception disabling
if(NOT RPP_ENABLE_EXCEPTIONS)
    if(
        CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
        CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    )
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")
    else()
        message(SEND_ERROR  "option  : RPP_ENABLE_EXCEPTIONS")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "disabling exceptions on this compiler unsupported")
    endif()
endif()

# handle exception disabling
if(NOT RPP_ENABLE_RTTI)
    if(
        CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
        CMAKE_CXX_COMPILER_ID STREQUAL "GNU"
    )
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
    else()
        message(SEND_ERROR  "option  : RPP_ENABLE_RTTI")
        message(SEND_ERROR  "compiler: ${CMAKE_CXX_COMPILER_ID}")
        message(FATAL_ERROR "disabling RTTI on this compiler unsupported")
    endif()
endif()
