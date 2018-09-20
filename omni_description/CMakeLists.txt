cmake_minimum_required(VERSION 2.8.3)
project(omni_description)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
  urdf
  xacro
  gazebo_plugins
  gazebo_ros
)

include_directories(
# include
  ${catkin_INCLUDE_DIRS}
)

file(GLOB xacro_files urdf/omni.urdf.xacro)
xacro_add_files(${xacro_files} TARGET media_files)

