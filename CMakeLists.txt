# Copyright (C) 2020 Istituto Italiano di Tecnologia (IIT). All rights reserved.
# This software may be modified and distributed under the terms of the Apache-2.0 license

cmake_minimum_required(VERSION 3.16)

set(OSQP_MATLAB_UPSTREAM_VERSION 0.6.2)
set(OSQP_MATLAB_CMAKE_REVISION 0)
set(OSQP_MATLAB_CMAKE_VERSION "${OSQP_MATLAB_UPSTREAM_VERSION}.${OSQP_MATLAB_CMAKE_REVISION}")
project(osqp-matlab-cmake-buildsystem
  LANGUAGES C CXX
  VERSION ${OSQP_MATLAB_CMAKE_VERSION})

include(GNUInstallDirs)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

option(BUILD_SHARED_LIBS "Build libraries as shared as opposed to static" ON)
option(OSQP_MATLAB_USES_MATLAB "Do you want to create the MATLAB bindings" ON)
option(OSQP_MATLAB_USES_OCTAVE "Do you want to create the Octave bindings" OFF)

set(OSQP_MATLAB_INSTALL_MATLAB_LIBDIR "mex" CACHE
    STRING "Location (relative to the install prefix) in which the Matlab mex libraries are installed.")
set(OSQP_MATLAB_INSTALL_MATLAB_MFILESDIR "mex" CACHE
    STRING "Location (relative to the install prefix) in which the Matlab .m files are installed.")
set(OSQP_MATLAB_INSTALL_OCTAVE_LIBDIR "octave" CACHE
    STRING "Location (relative to the install prefix) in which the Octave mex libraries are installed.")
set(OSQP_MATLAB_INSTALL_OCTAVE_MFILESDIR "octave" CACHE
    STRING "Location (relative to the install prefix) in which the Octave .m files are installed.")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/ycm-0.12)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/cmake-proposed)

# Enable RPATH support for installed binaries and libraries
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_FULL_BINDIR}"
  LIB_DIRS "${CMAKE_INSTALL_FULL_LIBDIR}"
  INSTALL_NAME_DIR "${CMAKE_INSTALL_FULL_LIBDIR}"
  USE_LINK_PATH)

# Encourage user to specify a build type (e.g. Release, Debug, etc.), otherwise set it to Release.
if(NOT CMAKE_CONFIGURATION_TYPES)
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
  endif()
endif()

## Find qdldl and osqp dependency
find_package(qdldl REQUIRED)
find_package(osqp REQUIRED)

## Clone osqp-matlab repository
## To use with some the osqp-matlab source coming from some other source, comment out
# the FetchContent* commands and set manually osqp-matlab_SOURCE_DIR
include(FetchContent)
# Temporary pointing to a fork, see https://github.com/oxfordcontrol/osqp-matlab/pull/34
FetchContent_Declare(
  osqp-matlab
  GIT_REPOSITORY https://github.com/traversaro/osqp-matlab
# GIT_TAG        v${OSQP_MATLAB_UPSTREAM_VERSION}
  GIT_TAG        aff21017e4396aed8cf7537a14fe540ae5b0c374
  )

FetchContent_MakeAvailable(osqp-matlab)

# Workaround for missing include https://github.com/oxfordcontrol/osqp-matlab/issues/27
file(DOWNLOAD "https://raw.githubusercontent.com/oxfordcontrol/osqp/v${OSQP_MATLAB_UPSTREAM_VERSION}/lin_sys/direct/qdldl/qdldl_interface.h" ${CMAKE_CURRENT_BINARY_DIR}/workaround_include/qdldl_interface.h)

# Common source files
set(M_FILES ${osqp-matlab_SOURCE_DIR}/osqp.m)
set(MEX_FILES ${osqp-matlab_SOURCE_DIR}/osqp_mex.hpp ${osqp-matlab_SOURCE_DIR}/osqp_mex.cpp)

if(OSQP_MATLAB_USES_MATLAB)
  find_package(Matlab REQUIRED)
  matlab_add_mex(
      NAME osqp_mex_matlab
      OUTPUT_NAME osqp_mex
      SRC ${MEX_FILES}
    LINK_TO ${Matlab_MEX_LIBRARY} ${Matlab_MX_LIBRARY} qdldl::qdldl osqp::osqp)
  # Workaround for missing include https://github.com/oxfordcontrol/osqp-matlab/issues/27
  target_include_directories(osqp_mex_matlab PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/workaround_include)
  install(
    TARGETS osqp_mex_matlab
    EXPORT ${PROJECT_NAME}
    DESTINATION ${OSQP_MATLAB_INSTALL_MATLAB_LIBDIR})
  install(
    FILES ${M_FILES}
    DESTINATION ${OSQP_MATLAB_INSTALL_MATLAB_MFILESDIR})
endif()

if(OSQP_MATLAB_USES_OCTAVE)
  find_package(Octave REQUIRED)

  add_library(osqp_mex_octave MODULE  ${MEX_FILES})
  set_target_properties(osqp_mex_octave PROPERTIES DEBUG_POSTFIX "")
  target_include_directories(osqp_mex_octave PUBLIC ${OCTAVE_INCLUDE_DIRS})
  target_link_libraries(osqp_mex_octave ${OCTAVE_LIBRARIES} qdldl::qdldl osqp::osqp)
  if(APPLE)
    set_target_properties(osqp_mex_octave PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
  endif()
  set_target_properties(osqp_mex_octave
    PROPERTIES OUTPUT_NAME osqp_mex
               PREFIX ""
               SUFFIX .mex)
  # Workaround for missing include https://github.com/oxfordcontrol/osqp-matlab/issues/27
  target_include_directories(osqp_mex_octave PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/workaround_include)
  install(
    TARGETS osqp_mex_octave
    EXPORT ${PROJECT_NAME}
    DESTINATION ${OSQP_MATLAB_INSTALL_OCTAVE_LIBDIR})
  install(
    FILES ${M_FILES}
    DESTINATION ${OSQP_MATLAB_INSTALL_OCTAVE_MFILESDIR})
endif()

include(AddUninstallTarget)
