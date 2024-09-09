cmake_minimum_required(VERSION 3.8)
project(bluespace_ai_xsens_mti_driver)

cmake_policy(SET CMP0057 NEW)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)

###########
## Build ##
###########

add_custom_target(BuildLib
    COMMAND ${CMAKE_COMMAND} -E echo "Building library in lib/xspublic"
    COMMAND ${CMAKE_COMMAND} -E chdir ${CMAKE_SOURCE_DIR}/lib/xspublic ${CMAKE_MAKE_PROGRAM}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/lib/xspublic
    COMMENT "Building library in lib/xspublic"
)

link_directories(${CMAKE_SOURCE_DIR}/lib/xspublic/xscontroller 
    ${CMAKE_SOURCE_DIR}/lib/xspublic/xscommon 
    ${CMAKE_SOURCE_DIR}/lib/xspublic/xstypes
)

add_executable(
    xsens_mti_node
    src/main.cpp
    src/xdainterface.cpp
    src/xdacallback.cpp
)

add_dependencies(xsens_mti_node BuildLib)

target_include_directories(xsens_mti_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/xspublic>
  $<INSTALL_INTERFACE:include>)

ament_target_dependencies(xsens_mti_node
  rclcpp
  tf2
  tf2_ros
  std_msgs
  geometry_msgs
  sensor_msgs
)

target_link_libraries(
    xsens_mti_node
    xscontroller
    xscommon
    xstypes
    pthread
    dl
)

install(TARGETS xsens_mti_node
  EXPORT export_${PROJECT_NAME}
  DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY launch param rviz urdf
  DESTINATION share/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
