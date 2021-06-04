cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(openfst VERSION 1.0)

include(FetchContent)
include(ExternalProject)
set(CMAKE_VERBOSE_MAKEFILE on)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

FetchContent_Declare(gflags
  URL      https://github.com/gflags/gflags/archive/v2.2.1.zip
  URL_HASH SHA256=4e44b69e709c826734dbbbd5208f61888a2faf63f239d73d8ba0011b2dccc97a
)
FetchContent_Declare(glog
  URL      https://github.com/google/glog/archive/v0.4.0.zip
  URL_HASH SHA256=9e1b54eb2782f53cd8af107ecf08d2ab64b8d0dc2b7f5594472f3bd63ca85cdc
)
FetchContent_MakeAvailable(gflags glog)

set(openfst_SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/openfst-src)
ExternalProject_Add(openfst
  URL               https://github.com/mjansche/openfst/archive/1.6.5.zip
  URL_HASH          SHA256=b720357a464f42e181d7e33f60867b54044007f50baedc8f4458a3926f4a5a78
  SOURCE_DIR        ${openfst_SOURCE_DIR}
  CONFIGURE_COMMAND ${openfst_SOURCE_DIR}/configure --prefix=${CMAKE_CURRENT_BINARY_DIR}
                      "CPPFLAGS=-I${gflags_BINARY_DIR}/include -I${glog_SOURCE_DIR}/src -I${glog_BINARY_DIR}"
                      "LDFLAGS=-L${gflags_BINARY_DIR} -L${glog_BINARY_DIR}"
                      "LIBS=-lgflags_nothreads -lglog -lpthread"
  BUILD_COMMAND     cp -r ${CMAKE_CURRENT_SOURCE_DIR}/patch/src ${openfst_SOURCE_DIR}
  COMMAND           make -j 4
)
add_dependencies(openfst gflags glog)

add_custom_command(TARGET openfst POST_BUILD
  COMMAND cp ${glog_SOURCE_DIR}/src/glog/log_severity.h ${glog_BINARY_DIR}/glog
  COMMAND cp -r ${gflags_BINARY_DIR}/include/gflags ${CMAKE_CURRENT_SOURCE_DIR}/libs/include
  COMMAND cp -r ${glog_BINARY_DIR}/glog ${CMAKE_CURRENT_SOURCE_DIR}/libs/include
  COMMAND cp -r ${openfst_SOURCE_DIR}/src/include/fst ${CMAKE_CURRENT_SOURCE_DIR}/libs/include
  COMMAND cp ${gflags_BINARY_DIR}/libgflags* ${CMAKE_CURRENT_SOURCE_DIR}/libs
  COMMAND cp ${glog_BINARY_DIR}/libglog* ${CMAKE_CURRENT_SOURCE_DIR}/libs
  COMMAND cp ${openfst_BINARY_DIR}/lib/libfst* ${CMAKE_CURRENT_SOURCE_DIR}/libs
)
