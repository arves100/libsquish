# cmake build file for squish
# by Stefan Roettger (stefan@stereofx.org)
# updated by Simon Brown (si@sjbrown.co.uk)

# features:
#   Xcode: builds universal binaries, uses SSE2 on i386 and Altivec on ppc
#   Unix and VS: SSE2 support is enabled by default
#   use BUILD_SQUISH_WITH_SSE2 and BUILD_SQUISH_WITH_ALTIVEC to override

cmake_minimum_required(VERSION 3.7)
project(squish)

option(BUILD_SHARED_LIBS "Build shared libraries." OFF)
option(BUILD_SQUISH_EXTRA "Build extra source code." OFF)

if ("${CMAKE_SYSTEM_NAME}" STREQUAL "ppc")
    set(NEW_CLFAGS -DSQUISH_USE_ALTIVEC=1)
    set(NEW_GCCFLAGS -maltivec)
else()
    set(NEW_CFLAGS -DSQUISH_USE_SSE=2)

    if (NOT MSVC)
        set(NEW_GCCFLAGS -msse2)
    endif()
endif()


set(SQUISH_HDRS
    squish.h
)

set(SQUISH_SRCS
    alpha.cpp
    alpha.h
    clusterfit.cpp
    clusterfit.h
    colourblock.cpp
    colourblock.h
    colourfit.cpp
    colourfit.h
    colourset.cpp
    colourset.h
    maths.cpp
    maths.h
    rangefit.cpp
    rangefit.h
    simd.h
    simd_float.h
    simd_sse.h
    simd_ve.h
    singlecolourfit.cpp
    singlecolourfit.h
    singlecolourlookup.inl
    squish.cpp
)

if (BUILD_SHARED_LIBS)
    set(LIBTYPE SHARED)
else()
    set(LIBTYPE STATIC)
endif()

add_library(squish ${LIBTYPE} ${SQUISH_SRCS} ${SQUISH_HDRS})
target_include_directories(squish PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_definitions(squish PRIVATE ${NEW_CFLAGS})
target_compile_options(squish PRIVATE ${NEW_GCCFLAGS})

IF (BUILD_SQUISH_EXTRA)
    set(SQUISHTEST_SRCS extra/squishtest.cpp)

    add_executable(squishtest ${SQUISHTEST_SRCS})
    set_target_properties(squishtest PROPERTIES DEBUG_POSTFIX "d")
    target_link_libraries(squishtest PRIVATE squish)

    set(SQUISHPNG_SRCS extra/squishpng.cpp)

    find_package(PNG)

    if (PNG_FOUND)
        set(CMAKE_PLATFORM_IMPLICIT_INCLUDE_DIRECTORIES)
        add_executable(squishpng ${SQUISHPNG_SRCS})
        set_target_properties(squishpng PROPERTIES DEBUG_POSTFIX "d")
        target_link_libraries(squishpng PRIVATE squish ${PNG_LIBRARIES})
        target_include_directories(squishpng PRIVATE ${PNG_INCLUDE_DIR})
    endif()
endif()

install(
    TARGETS squish
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    PUBLIC_HEADER DESTINATION include
)
