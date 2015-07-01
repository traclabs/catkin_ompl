# This was cobbled together by Ryan Luna to provide a current Catkin build of OMPL on ROS Hydro

cmake_minimum_required(VERSION 2.8)
if(NOT ${CMAKE_VERSION} VERSION_LESS 3.0.0)
    cmake_policy(SET CMP0042 OLD)
endif()
project(ompl_source)

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

add_subdirectory(ompl)
