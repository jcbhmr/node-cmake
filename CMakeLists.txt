cmake_minimum_required(VERSION 3.30)

project(node
  VERSION 23.8.0
  DESCRIPTION "TODO"
  HOMEPAGE_URL "https://github.com/jcbhmr/node-cmake"
  LANGUAGES C CXX
)

include(ExternalProject)
include(CTest)

if(CMAKE_HOST_WIN32 AND NOT WIN32)
  message(FATAL_ERROR "Cannot cross compile to another operating system on Windows")
endif()
if(WIN32 AND NOT CMAKE_HOST_WIN32)
  message(FATAL_ERROR "Cannot cross compile to Windows from another operating system")
endif()

if(BUILD_SHARED_LIBS)
  list(APPEND vcbuild_args "dll")
  list(APPEND configure_args "--shared")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  list(APPEND vcbuild_args "debug")
  list(APPEND configure_args "--debug")
elif(CMAKE_BUILD_TYPE STREQUAL "Release")
  list(APPEND vcbuild_args "release")
  list(APPEND configure_args "--release")
endif()

if(WIN32)
  ExternalProject_Add(node
    GIT_REPOSITORY https://github.com/nodejs/node.git
    GIT_TAG "v${PROJECT_VERSION}"
    GIT_SHALLOW TRUE
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE TRUE
    BUILD_COMMAND "./vcbuild" ${vcbuild_args}
    TEST_COMMAND "./vcbuild" test ${vcbuild_test_args}
    INSTALL_COMMAND ""
  )
else()
  find_program(MAKE_EXE NAMES gmake make nmake mingw32-make REQUIRED)
  find_program(NINJA_EXE NAMES ninja)
  if(NINJA_EXE)
    list(APPEND configure_args "--ninja")
  endif()
  ExternalProject_Add(node
    GIT_REPOSITORY https://github.com/nodejs/node.git
    GIT_TAG "v${PROJECT_VERSION}"
    GIT_SHALLOW TRUE
    CONFIGURE_COMMAND ./configure ${configure_args}
    BUILD_IN_SOURCE TRUE
    BUILD_COMMAND "${MAKE_EXE}" ${make_args}
    TEST_COMMAND "${MAKE_EXE}" test ${make_test_args}
    INSTALL_COMMAND "${MAKE_EXE}" install ${make_install_args}
  )
endif()
