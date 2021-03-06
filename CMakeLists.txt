cmake_minimum_required(VERSION 3.0)
project(ann)

# Set build type to RELEASE by default:
if (NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type provided, default to RELEASE.")
  set(CMAKE_BUILD_TYPE "RELEASE")
endif ()

option(BuildTest "BuildTest" ON)
#option(BuildBench "BuildBench" OFF)
option(BuildShared "BuildShared" OFF)
option(Sanitize "Sanitize" OFF)

if (BuildShared)
  set(LIB_TYPE SHARED)
else ()
  set(LIB_TYPE STATIC)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

find_package(CUDA REQUIRED)
set(CUDA_ARCH -gencode arch = compute_52, code = sm_52)
find_package(Threads REQUIRED)
#find_package(Boost)
find_package(LAPACK)
find_package(Thrust)
find_package(Math)

include_directories(. include ${Boost_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS})

set(CMAKE_CXX_STANDARD 11) 
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "-Wall -std=c++11 ${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "-g -pedantic -fno-omit-frame-pointer -fno-inline-functions")
if("${CMAKE_BUILD_TYPE}" STREQUAL "DEBUG")
  set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -G -Xcompiler -rdynamic -Wno-deprecated-gpu-targets")
endif()

if (Sanitize)
  set(CMAKE_CXX_FLAGS "-fsanitize=address -fsanitize=undefined ${CMAKE_CXX_FLAGS}")
endif ()

add_subdirectory(src)

if (BuildTest)
  enable_testing()
  add_subdirectory(test)
endif ()

if (BuildBench)
  add_subdirectory(bench)
endif ()

message(STATUS "")
message(STATUS "BUILD SUMMARY")
message(STATUS "  CMAKE_GENERATOR      : ${CMAKE_GENERATOR}")
message(STATUS "  Compiler ID          : ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "  Build type           : ${CMAKE_BUILD_TYPE}")
message(STATUS "  Library type         : ${LIB_TYPE}")
message(STATUS "  Build tests          : ${BuildTest}")
#message(STATUS "  Build benchmarks     : ${BuildBench}")
message(STATUS "  Sanitize flags       : ${Sanitize}")
message(STATUS "  Found CUDA            : ${CUDA_FOUND}")
message(STATUS "  FOUND LAPACKE         : ${LAPACK_FOUND}")
message(STATUS "  Boost include dirs   : ${Boost_INCLUDE_DIRS}")
message(STATUS "  CXX_FLAGS            : ${CMAKE_CXX_FLAGS}")
if ("${CMAKE_BUILD_TYPE}" STREQUAL "RELEASE")
  message(STATUS "  CXX_FLAGS_RELEASE    : ${CMAKE_CXX_FLAGS_RELEASE}")
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "DEBUG")
  message(STATUS "  CXX_FLAGS_DEBUG      : ${CMAKE_CXX_FLAGS_DEBUG}")
endif ()
message(STATUS "")
