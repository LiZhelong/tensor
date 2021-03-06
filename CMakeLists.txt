cmake_minimum_required(VERSION 3.1)

project(MatazureTensor CXX)

set(CMAKE_CXX_STANDARD 11)

find_package(CUDA 8.0)
if(CUDA_FOUND)
	option(WITH_CUDA "With CUDA" ON)
else()
	option(WITH_CUDA "With CUDA" OFF)
endif()

find_package(OpenMP)
if(OPENMP_FOUND)
	option(WITH_OPENMP "With OpenMP" ON)
else()
	option(WITH_OPENMP "With OpenMP" OFF)
endif()

option(BUILD_EXAMPLE "Build Example" ON)
option(BUILD_BENCHMARK "Build Benchmark" ON)
option(BUILD_TEST "Build Test" ON)

message("current compiler is ${CMAKE_CXX_COMPILER}")

include_directories(.)
set_property( GLOBAL PROPERTY USE_FOLDERS ON)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  # using Clang
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  # using GCC
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -xSSE3_ATOM -static-intel")
  # using Intel C++
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  # using Visual Studio C++
endif()

set(BUILD_SHARED_LIBS OFF)
if (MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")

	foreach (flag_var
					 CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
					 CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
		if (NOT BUILD_SHARED_LIBS AND NOT gtest_force_shared_crt)
			string(REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
		endif()
	endforeach()
endif()

if (WITH_OPENMP)
	find_package(OpenMP REQUIRED)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

if (UNIX)
	add_definitions("-x c++")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.2")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s -O3 -Wall -funroll-loops -ffast-math  ")

	if (ANDROID)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=atom -mtune=atom")
	endif()
endif()

if (WITH_CUDA)
	find_package(CUDA 8.0 REQUIRED)

	if(NOT(CMAKE_SIZEOF_VOID_P EQUAL 8))
		message(FATAL_ERROR "Matazure Tensor with CUDA Only support 64 bit platform")
	endif()

	#--expt-relaxed-constexpr  (don't use this)
	set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} --expt-extended-lambda -Wno-deprecated-gpu-targets")
	if (UNIX)
		set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} --std=c++11")
	endif()
	if (MSVC)
		set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -Xcompiler \"/wd4819 /wd4503\" ")
	endif()
endif()

message("CXX Compiler: ${CMAKE_CXX_COMPILER}")
message("CC Compiler: ${CMAKE_C_COMPILER}")

if (BUILD_EXAMPLE)
	add_subdirectory(example)
endif()

if (BUILD_BENCHMARK)
	add_subdirectory(benchmark)
endif()

if (BUILD_TEST)
	add_subdirectory(test)
endif()
