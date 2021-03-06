# Copyright 2018 The Cartographer Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 2.8.12)  # Ships with Ubuntu 14.04 (Trusty)

project(cartographer_mir)

set(PACKAGE_DEPENDENCIES
  cartographer_ros
  roscpp
  roslib
)

find_package(cartographer REQUIRED)
include("${CARTOGRAPHER_CMAKE_DIR}/functions.cmake")
google_initialize_cartographer_project()
google_enable_testing()

find_package(LuaGoogle REQUIRED)

find_package(catkin REQUIRED COMPONENTS ${PACKAGE_DEPENDENCIES})

# Override Catkin's GTest configuration to use GMock.
set(GTEST_FOUND TRUE)
set(GTEST_INCLUDE_DIRS ${GMOCK_INCLUDE_DIRS})
set(GTEST_LIBRARIES ${GMOCK_LIBRARIES})

catkin_package(CATKIN_DEPENDS message_runtime ${PACKAGE_DEPENDENCIES})

if (CATKIN_ENABLE_TESTING)
  catkin_add_gtest(configuration_files_test "cartographer_mir/configuration_files_test.cc")
  # catkin_add_gtest uses a plain (i.e. no PUBLIC/PRIVATE/INTERFACE) call to
  # target_link_libraries. That forces us to do the same.
  target_include_directories(configuration_files_test SYSTEM PUBLIC ${LUA_INCLUDE_DIR})
  target_link_libraries(configuration_files_test ${LUA_LIBRARIES})
  target_include_directories(configuration_files_test SYSTEM PUBLIC ${catkin_INCLUDE_DIRS})
  target_link_libraries(configuration_files_test ${catkin_LIBRARIES})
  add_dependencies(configuration_files_test ${catkin_EXPORTED_TARGETS})
  target_link_libraries(configuration_files_test cartographer)
  set(TARGET_COMPILE_FLAGS "${TARGET_COMPILE_FLAGS} ${GOOG_CXX_FLAGS}")
  set_target_properties(configuration_files_test PROPERTIES COMPILE_FLAGS ${TARGET_COMPILE_FLAGS})
endif()

install(DIRECTORY launch configuration_files urdf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
