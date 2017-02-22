cmake_minimum_required(VERSION 2.8)
if(NOT ${CMAKE_VERSION} VERSION_LESS 3.0.0)
  cmake_policy(SET CMP0042 OLD)
endif()
project(ompl CXX C)

find_package(catkin REQUIRED)

# set the default build type
if (NOT CMAKE_BUILD_TYPE)
  # By default, use Release mode
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Type of build" FORCE)

  # On 32bit architectures, use RelWithDebInfo
  if (CMAKE_COMPILER_IS_GNUCC AND CMAKE_SIZEOF_VOID_P EQUAL 4)
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Type of build" FORCE)
  endif()
endif()


# This shouldn't be necessary, but there has been trouble
# with MSVC being set off, but MSVCXX ON.
if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/ompl/CMakeModules")
include(GNUInstallDirs)
include(CompilerSettings)
include(OMPLVersion)
include(OMPLUtils)

# OMPL include path
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/ompl/src")

if(MSVC)
  add_definitions(-DBOOST_ALL_NO_LIB)
  add_definitions(-DBOOST_PROGRAM_OPTIONS_DYN_LINK)
endif(MSVC)
if(IS_ICPC)
  set(Boost_USE_STATIC_LIBS ON CACHE STRING "Use statically linked Boost libraries")
else(IS_ICPC)
  # Ensure dynamic linking with boost unit_test_framework
  add_definitions(-DBOOST_TEST_DYN_LINK)
endif(IS_ICPC)

# Do one quiet find_package on Boost to see if it is recent enough to
# have the try_join_for call
find_package(Boost QUIET 1.50)

# If Boost is not found at all, the check for version 1.50 below will die in
# the most ungraceful manner because Boost_VERSION is not defined.  Define the
# variable here for a more useful error.
if (NOT ${Boost_FOUND})
  set(Boost_VERSION 0)
endif()

# try_join_for requires the chrono library, so if we will use
# try_join_for, we need to include the chrono component
# Must recheck the Boost version, since update_bindings will re-run CMake
# and this will pass for versions of Boost < 1.50
if (${Boost_FOUND} AND ${Boost_VERSION} GREATER 104900) # Boost version is at least 1.50
  # we're using chrono
  find_package(Boost COMPONENTS date_time thread serialization filesystem system program_options unit_test_framework chrono REQUIRED)
else()
  # don't use chrono
  find_package(Boost COMPONENTS date_time thread serialization filesystem system program_options unit_test_framework REQUIRED)
endif()
include_directories(SYSTEM ${Boost_INCLUDE_DIR})

if(${Boost_VERSION} LESS 105300)
  # Include bundled version of boost::odeint if it isn't installed natively
  set(ODEINT_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/ompl/src/external")
  include_directories(SYSTEM "${ODEINT_INCLUDE_DIR}")
endif()

if(APPLE AND CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  include(GetPrerequisites)
  get_prerequisites("${Boost_SYSTEM_LIBRARY}" _libs 0 0 "/" "")
  set(CXXSTDLIB "")
  foreach(_lib ${_libs})
    if(_lib MATCHES "libc\\+\\+")
      set(CXXSTDLIB "libc++")
    elseif(_lib MATCHES "libstdc\\+\\+")
      set(CXXSTDLIB "libstdc++")
    endif()
  endforeach()
  if(CXXSTDLIB)
    add_definitions(-stdlib=${CXXSTDLIB})
  endif()
endif()

install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/ompl/doc/markdown/FindOMPL.cmake"
  DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/ompl${OMPL_INSTALL_SUFFIX}"
  COMPONENT ompl
  RENAME ompl-config.cmake)

# pthread is sometimes needed, depending on OS / compiler
find_package(Threads QUIET)

# Barebones compilation only
set(OMPL_EXTENSION_MORSE 0)
set(OMPL_EXTENSION_OPENDE 0)
set(OMPL_EXTENSION_TRIANGLE 0)
option(OMPL_REGISTRATION OFF)

# CATKIN stuff
catkin_package(
  LIBRARIES
  ompl
  INCLUDE_DIRS
  ompl/src
  )

# Explicitly skipping the root level CMakeLists.txt in ompl.
# There is a conflict between catkin and ompl when setting the
# library output directory that causes a compilation error.
# Also, this allows us to skip compilation of tests, demos,
# and other superfluous code.
add_subdirectory(ompl/src)
