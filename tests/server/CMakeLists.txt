##
##  Copyright 2021 Alibaba, Inc. and its affiliates. All Rights Reserved.
##
##  Licensed under the Apache License, Version 2.0 (the "License");
##  you may not use this file except in compliance with the License.
##  You may obtain a copy of the License at
##  
##      http://www.apache.org/licenses/LICENSE-2.0
##  
##  Unless required by applicable law or agreed to in writing, software
##  distributed under the License is distributed on an "AS IS" BASIS,
##  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
##  See the License for the specific language governing permissions and
##  limitations under the License.
##
##  \author   hechong.xyf
##  \date     Feb 2021
##  \brief    Detail cmake build script
##

include(../../cmake/bazel.cmake)
include(../../cmake/option.cmake)

if(APPLE)
  set(APPLE_FRAMEWORK_LIBS
      -framework CoreFoundation
      -framework CoreGraphics
      -framework CoreData
      -framework CoreText
      -framework Security
      -framework Foundation
      -Wl,-U,_MallocExtension_ReleaseFreeMemory
      -Wl,-U,_ProfilerStart
      -Wl,-U,_ProfilerStop
      -Wl,-U,_RegisterThriftProtocol
    )
endif()

git_version(PROXIMA_BE_VERSION ${CMAKE_CURRENT_SOURCE_DIR})
file(GLOB ALL_TEST_SRCS *_test.cc)
foreach (CC_SRCS ${ALL_TEST_SRCS})
  get_filename_component(CC_TARGET ${CC_SRCS} NAME_WE)
  cc_gmock(
      NAME ${CC_TARGET} STRICT
      LIBS proxima_be_server
           proxima_be_agent
           proxima_be_index
           proxima_be_meta
           proxima_be_common
           proxima_be_proto
           proxima_search_client_static
           proxima
           ${CMAKE_THREAD_LIBS_INIT}
           ${CMAKE_DL_LIBS}
      SRCS ${CC_SRCS}
      INCS .. ../../sdk/cpp/include/ ../../src
      LDFLAGS ${APPLE_FRAMEWORK_LIBS}
      VERSION "${PROXIMA_BE_VERSION}"
    )
  set_target_properties(${CC_TARGET} PROPERTIES INSTALL_RPATH ${LIB_PATH})
  cc_test_suite(proxima_be_server ${CC_TARGET})
endforeach ()
