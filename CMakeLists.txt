# USING cmake:
#   mkdir build
#   cd build
#   cmake ..
#   make -j

# RUN unit tests:
#   cd build
#   ctest



# OBJECT library requires 2.8.8
cmake_minimum_required(VERSION 2.8.8 FATAL_ERROR)
project(ch_constructor CXX)

enable_testing()

include(CMakeDetermineCXXCompiler)

# default to RelWithDebInfo
if("${CMAKE_BUILD_TYPE}" STREQUAL "")
	set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		set(EXTRA_CXX_CLANG_FLAGS "-Wshorten-64-to-32")
	endif()
	set(EXTRA_CXX_FLAGS "-Wall -Wextra -Wno-unused-parameter -pedantic ${EXTRA_CXX_CLANG_FLAGS}" CACHE STRING "Extra flags used by the compiler during all build types.")
	set(EXTRA_EXE_LINKER_FLAGS "-Wl,--as-needed" CACHE STRING "Extra flags used by the linker.")
	set(EXTRA_EXE_LINKER_FLAGS_RELEASE "-flto" CACHE STRING "Extra flags used by the linker for the Release build type.")
	set(EXTRA_EXE_LINKER_FLAGS_RELWITHDEBINFO "-flto" CACHE STRING "Extra flags used by the linker for the RelWithDebInfo build type.")
else()
	set(EXTRA_CXX_FLAGS "" CACHE STRING "Extra flags used by the compiler during all build types.")
	set(EXTRA_EXE_LINKER_FLAGS "" CACHE STRING "Extra flags used by the linker.")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EXTRA_CXX_FLAGS} -std=c++11 -fopenmp")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${EXTRA_EXE_LINKER_FLAGS} -fopenmp")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} ${EXTRA_EXE_LINKER_FLAGS_RELEASE}")
set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO} ${EXTRA_EXE_LINKER_FLAGS_RELWITHDEBINFO}")

option(VERBOSE "Verbose logging" OFF)

if(NOT VERBOSE)
	add_definitions(-DNVERBOSE)
endif()

option(LARGE_GRAPH "Large graphs with more than 2**32 nodes/edges" OFF)
if(LARGE_GRAPH)
	add_definitions(-DLARGE_GRAPH)
endif()

# compile shared sources only once, and reuse object files in both,
# as they are compiled with the same options anyway
add_library(common OBJECT
	src/nodes_and_edges.cpp
	src/file_formats.cpp
)

add_executable(ch_constructor
	src/ch_constructor.cpp
	$<TARGET_OBJECTS:common>
)

add_executable(run_tests
	src/run_tests.cpp
	src/unit_tests.cpp
	$<TARGET_OBJECTS:common>
)

add_test(NAME unit-test
	WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/src"
	COMMAND $<TARGET_FILE:run_tests>
)
