cmake_minimum_required(VERSION 2.8)

project(BAT)

set(BAT_VERSION_MAJOR 1)
set(BAT_VERSION_MINOR 1)
set(BAT_VERSION_PATCH 0)
set(BAT_VERSION_TWEAK "-cmake")
set(BAT_VERSION "${BAT_VERSION_MAJOR}.${BAT_VERSION_MINOR}.${BAT_VERSION_PATCH}")

set(PROJECT_VERSION_MAJOR ${BAT_VERSION_MAJOR})
set(PROJECT_VERSION_MINOR ${BAT_VERSION_MINOR})
set(PROJECT_VERSION_PATCH ${BAT_VERSION_PATCH})
set(PROJECT_VERSION_TWEAK ${BAT_VERSION_TWEAK})
set(PROJECT_VERSION ${BAT_VERSION})

add_definitions(-DVERSION="${BAT_VERSION}")

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing -Wpedantic -Wall -Wextra -Werror")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fPIC")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--as-needed")

option(ENABLE_PARALLELIZATION "enable the multi-thread parallelization. Default: off" OFF)
if (ENABLE_PARALLELIZATION)
  find_package(OpenMP)
  add_definitions(-DTHREAD_PARALLELIZATION=1)
else (ENABLE_PARALLELIZATION)
  add_definitions(-DTHREAD_PARALLELIZATION=0)
endif (ENABLE_PARALLELIZATION)

option(BAT_WITH_CUBA "compile with Cuba support. Default: off" OFF)
if (BAT_WITH_CUBA)
  include(ExternalProject)
  set(CUBA_DIR ${CMAKE_CURRENT_BINARY_DIR}/deps/cuba)
  ExternalProject_Add(Cuba
    PREFIX ${CUBA_DIR}
    URL "http://www.feynarts.de/cuba/Cuba-4.2.tar.gz"
    URL_MD5 "a3eb0751d4872a3f86401694bc5bedae"
    TIMEOUT 300
    CONFIGURE_COMMAND ${CUBA_DIR}/src/Cuba/configure --prefix=${CUBA_DIR} CFLAGS=-fPIC\ -O3\ -fomit-frame-pointer\ -ffast-math\ -Wall
    BUILD_COMMAND make lib
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND make install
    )
  add_library(cuba STATIC IMPORTED)
  set_target_properties(cuba PROPERTIES IMPORTED_LOCATION ${CUBA_DIR}/lib)
  add_definitions(-DHAVE_CUBA_H=1 -DCUBAVERSION=42)
endif (BAT_WITH_CUBA)


# require root
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})
find_package(ROOT)
include_directories(${ROOT_INCLUDE_DIR})

# tweak root version
string (REGEX REPLACE "(\\.|/)" "0" ROOTVERSION ${ROOT_VERSION})
add_definitions(-DROOTVERSION=${ROOTVERSION})

#message("Find ROOT version: " ${ROOT_VERSION})
#message("Enabled ROOT libraries: " ${ROOT_LIBRARIES})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/BAT)

add_subdirectory(src)
add_subdirectory(models/base)
add_subdirectory(models/mtf)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
