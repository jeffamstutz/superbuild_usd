## Copyright 2021 Jefferson Amstutz
## SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.16)

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX
      "${CMAKE_BINARY_DIR}/install"
      CACHE STRING "Final install location." FORCE)
endif()

set(CMAKE_BUILD_TYPE_INIT "Release")

project(build_usd)

find_package(Git REQUIRED)

include(ExternalProject)
include(ProcessorCount)
include(CMakePrintHelpers)
include(macros.cmake)

ProcessorCount(PROCESSOR_COUNT)
set(NUM_BUILD_JOBS ${PROCESSOR_COUNT} CACHE STRING "Number of build jobs '-j <n>'")

set(DEFAULT_BUILD_COMMAND cmake --build . --config release -j ${NUM_BUILD_JOBS})
option(GET_TBB "Download TBB during build process, otherwise TBB_ROOT must be set" ON)

get_filename_component(INSTALL_DIR_ABSOLUTE
  ${CMAKE_INSTALL_PREFIX} ABSOLUTE BASE_DIR ${CMAKE_CURRENT_BINARY_DIR})

## Build projects ##

if (GET_TBB)
  include(get_tbb.cmake)
  set(TBB_TARGET tbb)
else()
  if (NOT DEFINED TBB_ROOT)
    message(FATAL_ERRROR "TBB_ROOT must be specified if GET_TBB=OFF")
  endif()
endif()

build_subproject(
  NAME Blosc
  URL "https://github.com/Blosc/c-blosc/archive/refs/tags/v1.5.4.zip"
  BUILD_ARGS
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
)

build_subproject(
  NAME OpenVDB
  URL "https://github.com/AcademySoftwareFoundation/openvdb/archive/refs/tags/v6.1.0.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_ROOT}
    -DTbb_INCLUDE_DIR=${TBB_ROOT}/include
    -DTbb_tbb_LIBRARY=${TBB_ROOT}/lib/intel64/gcc4.8/libtbb.so
    -DOPENVDB_BUILD_BINARIES=OFF
    -DOPENVDB_BUILD_PYTHON_MODULE=OFF
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  DEPENDS_ON Blosc ${TBB_TARGET}
)

build_subproject(
  NAME USD
  URL "https://github.com/PixarAnimationStudios/USD/archive/refs/tags/v21.08.zip"
  PATCH_FILE ${CMAKE_SOURCE_DIR}/usd_build_fix.patch
  BUILD_ARGS
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    -DPXR_BUILD_EXAMPLES=OFF
    -DPXR_BUILD_IMAGING=OFF
    -DPXR_BUILD_TESTS=OFF
    -DPXR_BUILD_TUTORIALS=OFF
    -DPXR_BUILD_USDVIEW=OFF
    -DPXR_BUILD_USD_IMAGING=OFF
    -DPXR_BUILD_USD_TOOLS=OFF
    -DPXR_ENABLE_GL_SUPPORT=OFF
    -DPXR_ENABLE_HDF5_SUPPORT=OFF
    -DPXR_ENABLE_OPENVDB_SUPPORT=ON
    -DPXR_ENABLE_MATERIALX_SUPPORT=OFF
    -DPXR_ENABLE_PTEX_SUPPORT=OFF
    -DPXR_USE_PYTHON_3=ON
    -DTBB_ROOT=${TBB_ROOT}
    -DTBB_INCLUDE_DIR=${TBB_ROOT}/include
    -DTBB_tbb_LIBRARY_DEBUG=${TBB_ROOT}/lib/intel64/gcc4.8/libtbb_debug.so
    -DTBB_tbb_LIBRARY_RELEASE=${TBB_ROOT}/lib/intel64/gcc4.8/libtbb.so
  DEPENDS_ON OpenVDB
)
