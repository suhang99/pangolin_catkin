cmake_minimum_required(VERSION 2.8.3)
project(pangolin_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

option(PANGOLIN_USE_GUI "Build 3rdparty pangolin package with GUI support" ON)
option(PANGOLIN_USE_VIDEO "Build 3rdparty pangolin package with Video support" ON)
option(PANGOLIN_USE_FFMPEG "Build pangolin with ffmpeg backend (broken on newer ffmpeg; use only if you have an older ffmpeg toolchain)" OFF)
option(PANGOLIN_USE_OCULUS "Build 3rdparty pangolin package with Oculus support" OFF)
option(PANGOLIN_USE_VARS "Build 3rdparty pangolin package with Vars support" ON)

find_package(FREEGLUT QUIET)
find_package(GLUT QUIET)

set(PANGOLIN_CMAKE_OPTIONS "")
if (PANGOLIN_USE_GUI)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_GUI:BOOL=ON")
  find_package(OpenGL REQUIRED QUIET)
  find_package(GLEW REQUIRED QUIET)
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_GUI:BOOL=OFF")
endif()

if (PANGOLIN_USE_VIDEO)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VIDEO:BOOL=ON")
  find_package(DepthSense)
  if (PANGOLIN_USE_FFMPEG)
    list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_FFMPEG:BOOL=ON")
  else()
    # Pangolin 0.6 uses deprecated ffmpeg APIs that no longer compile with current ffmpeg releases.
    list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_FFMPEG:BOOL=OFF")
  endif()
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VIDEO:BOOL=OFF")
endif()

if (PANGOLIN_USE_OCULUS)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_OCULUS:BOOL=ON")
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_OCULUS:BOOL=OFF")
endif()

if (PANGOLIN_USE_VARS)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VARS:BOOL=ON")
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VARS:BOOL=OFF")
endif()

if (CMAKE_BUILD_TYPE)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DCMAKE_BUILD_TYPE:String=${CMAKE_BUILD_TYPE}")
endif()

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)
set(CATKIN_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})

set(Pangolin_Version 0.6)
ExternalProject_Add(pangolin_EXTERNAL
  URL https://github.com/stevenlovegrove/Pangolin/archive/v${Pangolin_Version}.zip
  UPDATE_COMMAND ""
  CMAKE_ARGS 
    -DBUILD_SHARED_LIBS:BOOL=ON
    ${PANGOLIN_CMAKE_OPTIONS}
    -DBUILD_EXAMPLES:BOOL=OFF
    -DBUILD_TESTS:BOOL=OFF
    -DCMAKE_CXX_FLAGS=-include\ cstdint\ -include\ limits
  BUILD_COMMAND ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_DEVEL_PREFIX} . && make install -j 8
  INSTALL_COMMAND ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_INSTALL_PREFIX} . && make install -j 8
)

cs_add_library(${PROJECT_NAME} src/empty.cpp)
add_dependencies(${PROJECT_NAME} pangolin_EXTERNAL)

if(EXISTS "${CATKIN_DEVEL_PREFIX}/lib/libpangolin${CMAKE_SHARED_LIBRARY_SUFFIX}")
  SET(PANGOLIN_FOUND TRUE)
else()
  SET(PANGOLIN_FOUND FALSE)
endif()

cs_install()
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS pangolin-extras.cmake
  )
