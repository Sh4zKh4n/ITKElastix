cmake_minimum_required(VERSION 3.10.2)
project(Elastix)

# To ease enablement with Python packaging
if(DEFINED ENV{ELASTIX_USE_OPENCL})
  set(ELASTIX_USE_OPENCL ON CACHE BOOL "Enable OpenCL support in Elastix")
endif()

set(Elastix_LIBRARIES elastix_lib transformix_lib)
if(ELASTIX_USE_OPENCL)
  list(APPEND Elastix_LIBRARIES elxOpenCL)
endif()
set(ELASTIX_BUILD_EXECUTABLE OFF)
set(ELASTIX_BUILD_EXECUTABLE OFF CACHE BOOL "Generate executable or library")
# Avoid LGPL code and ANN shared library
if(NOT DEFINED USE_KNNGraphAlphaMutualInformationMetric)
  set(USE_KNNGraphAlphaMutualInformationMetric OFF)
  set(USE_KNNGraphAlphaMutualInformationMetric OFF CACHE BOOL "Use KNN metric.  Requires ANN library.")
endif()

if(CMAKE_COMPILER_IS_GNUCXX AND
  ("${CMAKE_CXX_COMPILER_VERSION}" VERSION_EQUAL "4.8") OR
  ("${CMAKE_CXX_COMPILER_VERSION}" VERSION_GREATER "4.8" AND "${CMAKE_CXX_COMPILER_VERSION}" VERSION_LESS "5.0") )
  set(ELASTIX_USE_OPENMP OFF CACHE BOOL "Use OpenMP in elastix")
endif()
if(CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-aggressive-loop-optimizations")
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(NOT ITK_SOURCE_DIR)
  find_package(ITK REQUIRED)
endif()

include(FetchContent)

set(_itk_build_testing ${BUILD_TESTING})
#set(BUILD_TESTING OFF)
set(_itk_build_shared ${BUILD_SHARED_LIBS})
set(BUILD_SHARED_LIBS OFF)
set(elastix_GIT_REPOSITORY "https://github.com/SuperElastix/elastix.git")
set(elastix_GIT_TAG "1f973ae9bc2ef551f984ad7df35167e266e55ee0")
FetchContent_Declare(
  elx
  GIT_REPOSITORY ${elastix_GIT_REPOSITORY}
  GIT_TAG ${elastix_GIT_TAG})
FetchContent_GetProperties(elx)
if(NOT elx_POPULATED)
  FetchContent_Populate(elx)
  # Use CMake's FindOpenCL.cmake, which is backend agnostic
  file(REMOVE ${elx_SOURCE_DIR}/CMake/FindOpenCL.cmake)
  if(ELASTIX_USE_OPENCL)
    find_package(OpenCL REQUIRED)
    set(OPENCL_INCLUDE_DIRS ${OpenCL_INCLUDE_DIRS} CACHE PATH "OpenCL include directories")
    set(OPENCL_LIBRARIES ${OpenCL_LIBRARIES} CACHE FILEPATH "OpenCL library")
  endif()
  add_subdirectory(${elx_SOURCE_DIR} ${elx_BINARY_DIR})
endif()
set(Elastix_DIR "${elx_BINARY_DIR}")
find_package(Elastix REQUIRED)
include_directories( ${ELASTIX_INCLUDE_DIRS} )
link_directories( ${ELASTIX_LIBRARY_DIRS} )
#include(${ELASTIX_USE_FILE})
if(ELASTIX_USE_OPENMP)
  find_package(OpenMP QUIET)
  if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${OpenMP_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} ${OpenMP_STATIC_LINKER_FLAGS}")
    add_definitions(-DELASTIX_USE_OPENMP)
  endif()
endif()
if(ELASTIX_USE_OPENCL)
  add_definitions(-DELASTIX_USE_OPENCL)
endif()
set(BUILD_TESTING ${_itk_build_testing})
set(BUILD_SHARED_LIBS ${_itk_build_shared})


if(NOT ITK_SOURCE_DIR)
  find_package(ITK REQUIRED)
  list(APPEND CMAKE_MODULE_PATH ${ITK_CMAKE_DIR})
  include(ITKModuleExternal)
else()
  set(ITK_DIR ${CMAKE_BINARY_DIR})
  itk_module_impl()
endif()
