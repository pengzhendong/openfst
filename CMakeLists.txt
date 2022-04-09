cmake_minimum_required(VERSION 3.16)
project(openfst VERSION 1.0)

include(FetchContent)
include(ExternalProject)
set(CMAKE_VERBOSE_MAKEFILE on)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

FetchContent_Declare(gflags
  URL https://github.com/gflags/gflags/archive/v2.2.2.zip
)
FetchContent_Declare(glog
  URL https://github.com/google/glog/archive/v0.6.0.zip
)
option(BUILD_SHARED_LIBS "Build glog shared libraries" OFF)
FetchContent_MakeAvailable(gflags glog)

set(openfst_SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/openfst-src)
ExternalProject_Add(openfst
  URL               https://github.com/mjansche/openfst/archive/1.8.0.zip
  SOURCE_DIR        ${openfst_SOURCE_DIR}
  CONFIGURE_COMMAND ${openfst_SOURCE_DIR}/configure --prefix=${CMAKE_CURRENT_BINARY_DIR}
                      "CPPFLAGS=-I${gflags_BINARY_DIR}/include -I${glog_SOURCE_DIR}/src -I${glog_BINARY_DIR}"
                      "LDFLAGS=-L${gflags_BINARY_DIR} -L${glog_BINARY_DIR}"
                      "LIBS=-lgflags_nothreads -lglog -lpthread"
  BUILD_COMMAND     cp -r ${CMAKE_CURRENT_SOURCE_DIR}/patch/src ${openfst_SOURCE_DIR}
  COMMAND           make -j 4
)
add_dependencies(openfst gflags glog)
