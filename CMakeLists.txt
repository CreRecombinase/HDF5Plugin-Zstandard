cmake_minimum_required(VERSION 3.15)
project(zstd_hdf5 C)
set(default_build_type "Release")
include(GNUInstallDirs)
# options
# set(PLUGIN_INSTALL_PATH "/usr/local/hdf5/lib/plugin" CACHE PATH
#       "Where to install the dynamic HDF5-plugin")

# sources

SET(CMAKE_SKIP_BUILD_RPATH  FALSE)
# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
IF("${isSystemDir}" STREQUAL "-1")
   SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
ENDIF("${isSystemDir}" STREQUAL "-1")


add_library(zstd_h5 SHARED zstd_h5plugin.c zstd_h5plugin.h)
set_target_properties(zstd_h5 PROPERTIES PUBLIC_HEADER "zstd_h5plugin.h")


# dependencies
# if(MSVC)
#     # FindHDF5.cmake does not find Windows installations. Try to
#     # use an environment variable instead until the official "find"
#     # file can be updated for Windows.
#     #
#     # Note that you have to set this environment variable by hand.
#     file(TO_CMAKE_PATH "$ENV{HDF5_DIR}" HDF5_HINT)
#     set(HDF5_DIR ${HDF5_HINT} CACHE STRING "Path to HDF5 CMake config directory.")
#     find_package(HDF5 REQUIRED HINTS ${HDF5_DIR})
# else(MSVC)
    find_package(HDF5 REQUIRED)
    # endif(MSVC)
  message(STATUS "HDF5 INCLUDE: ${HDF5_INCLUDE_DIRS}")
#target_include_directories()

# HDF5 plugin as shared library

target_link_libraries(zstd_h5 zstd ${HDF5_LIBRARIES})
target_include_directories(zstd_h5 PUBLIC ${HDF5_INCLUDE_DIRS})
set(DEST_DIR "${CMAKE_INSTALL_PREFIX}")
install(TARGETS zstd_h5 EXPORT zstdh5
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

install(EXPORT zstdh5 FILE zstdh5.cmake NAMESPACE zstdh5:: DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zstdh5)
