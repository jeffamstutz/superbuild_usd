## Copyright 2021 Jefferson Amstutz
## SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.16)

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX
      "${CMAKE_BINARY_DIR}/install"
      CACHE STRING "Final install location." FORCE)
endif()

project(build_usd)

find_package(Git REQUIRED)

include(ExternalProject)
include(ProcessorCount)
include(macros.cmake)

ProcessorCount(PROCESSOR_COUNT)
set(NUM_BUILD_JOBS ${PROCESSOR_COUNT} CACHE STRING "Number of build jobs '-j <n>'")

set(DEFAULT_BUILD_COMMAND cmake --build . --config release -j ${NUM_BUILD_JOBS})

get_filename_component(INSTALL_DIR_ABSOLUTE
  ${CMAKE_INSTALL_PREFIX} ABSOLUTE BASE_DIR ${CMAKE_CURRENT_BINARY_DIR})

## Build projects ##

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
    -DOPENVDB_BUILD_BINARIES=OFF
    -DOPENVDB_BUILD_PYTHON_MODULE=OFF
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  DEPENDS_ON Blosc
)

build_subproject(
  NAME MaterialX
  URL "https://github.com/materialx/MaterialX/releases/download/v1.38.2/MaterialX-1.38.2.zip"
  BUILD_ARGS
    -DMATERIALX_BUILD_RENDER=OFF
    -DMATERIALX_BUILD_TESTS=OFF
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
    -DPXR_ENABLE_MATERIALX_SUPPORT=ON
    -DPXR_ENABLE_PTEX_SUPPORT=OFF
    -DPXR_USE_PYTHON_3=ON
  DEPENDS_ON OpenVDB MaterialX
)
