cmake_minimum_required(VERSION 2.8.3)
project(avexis_gazebo)
set(CMAKE_CXX_FLAGS "-std=c++0x ${CMAKE_CXX_FLAGS}")
find_package(catkin REQUIRED COMPONENTS
  gazebo_ros
  urdf
  control_toolbox
  rostime
  message_generation
)

find_package(gazebo REQUIRED)
find_package(Eigen3)

add_message_files(
    FILES
    BodySetpoint.msg
)

# build messages
generate_messages(
    DEPENDENCIES geometry_msgs
    )

catkin_package(
 INCLUDE_DIRS include
 LIBRARIES ${PROJECT_NAME} 
 DEPENDS
    gazebo_ros
    control_toolbox
    rostime
)

link_directories(${GAZEBO_LIBRARY_DIRS})
include_directories(include ${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS} ${GAZEBO_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS})
add_definitions(${EIGEN3_DEFINITIONS})

# world plugin to simulate buoyancy and viscous force
add_library(avexis_gazebo_fluid src/avexis_gazebo_fluid.cpp include/avexis_gazebo/avexis_gazebo_fluid.h)
target_link_libraries(avexis_gazebo_fluid ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})
add_dependencies(avexis_gazebo_fluid avexis_gazebo_generate_messages_cpp)

# model plugin subscribes to body and joint efforts and applies them in Gazebo
add_library(avexis_gazebo_control src/avexis_gazebo_control.cpp include/avexis_gazebo/avexis_gazebo_control.h)
target_link_libraries(avexis_gazebo_control ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})
add_dependencies(avexis_gazebo_control avexis_gazebo_generate_messages_cpp)

# node to perform PID control for body (thrusters) and joints
add_executable(pid_control src/avexis_pids_main.cpp
src/avexis_pids.cpp include/avexis_gazebo/avexis_pids.h
src/avexis_pids_body.cpp include/avexis_gazebo/avexis_pids_body.h
src/avexis_pids_joint.cpp include/avexis_gazebo/avexis_pids_joint.h)
target_link_libraries(pid_control ${catkin_LIBRARIES})
add_dependencies(pid_control avexis_gazebo_generate_messages_cpp)


