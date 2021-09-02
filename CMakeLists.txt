cmake_minimum_required(VERSION 3.0.2)
project(conveyor_gazebo_sim)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

# This package was developed with gazebo7, but now uses gazebo8.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra")
endif()

find_package(catkin REQUIRED COMPONENTS
  gazebo_ros
  gazebo_plugins
  roscpp
  std_msgs
  std_srvs
  tf
  geometry_msgs
  message_generation
)

find_package(gazebo REQUIRED)

########################
## Message generation ##
########################

add_message_files(DIRECTORY msg
  FILES
  ConveyorBeltState.msg
)

add_service_files(DIRECTORY srv
  FILES
  ConveyorBeltControl.srv
)

generate_messages(
  DEPENDENCIES
  geometry_msgs
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS
  message_runtime
  std_srvs
  tf
)

###########
## Build ##
###########

include_directories(
  include
  ${GAZEBO_INCLUDE_DIRS}
  ${roscpp_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

link_directories(
  ${GAZEBO_LIBRARY_DIRS}
)

# Create the libConveyorBeltPlugin.so library.
set(conveyor_belt_plugin_name ConveyorBeltPlugin)
add_library(${conveyor_belt_plugin_name} src/ConveyorBeltPlugin.cc)
target_link_libraries(${conveyor_belt_plugin_name}
  ${GAZEBO_LIBRARIES}
  ${side_contact_plugin_name}
)
install(TARGETS ${conveyor_belt_plugin_name}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Create the libROSConveyorBeltPlugin.so library.
set(ros_conveyor_belt_plugin_name ROSConveyorBeltPlugin)
add_library(${ros_conveyor_belt_plugin_name} src/ROSConveyorBeltPlugin.cc)
target_link_libraries(${ros_conveyor_belt_plugin_name}
  ${conveyor_belt_plugin_name}
  ${GAZEBO_LIBRARIES}
)
add_dependencies(${ros_conveyor_belt_plugin_name}
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
)
install(TARGETS ${ros_conveyor_belt_plugin_name}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# # Create the libConveyorCongestionPlugin.so library.
# set(conveyor_congestion_plugin ConveyorCongestionPlugin)
# add_library(${conveyor_congestion_plugin} src/ConveyorCongestionPlugin.cc)
# target_link_libraries(${conveyor_congestion_plugin}
#   ${GAZEBO_LIBRARIES}
# )
# add_dependencies(${conveyor_congestion_plugin}
#   ${${PROJECT_NAME}_EXPORTED_TARGETS}
# )
# install(TARGETS ${conveyor_congestion_plugin}
#   ARCHIVE DESTINATION lib
#   LIBRARY DESTINATION lib
#   RUNTIME DESTINATION bin
# )