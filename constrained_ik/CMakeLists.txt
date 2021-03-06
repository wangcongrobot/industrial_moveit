cmake_minimum_required(VERSION 2.8.12)
project(constrained_ik)

if(CMAKE_VERSION VERSION_LESS 3.1)
    add_compile_options(-std=c++11)
else()
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS ON)
endif()

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
              kdl_parser
              roscpp
              urdf
              eigen_conversions
              moveit_ros_planning
              moveit_core
              dynamic_reconfigure
              tf_conversions
              cmake_modules
              industrial_collision_detection)

## System dependencies are found with CMake's conventions
find_package(orocos_kdl REQUIRED)
find_package(Boost REQUIRED)
find_package(Eigen REQUIRED)

#add dynamic reconfigure api
generate_dynamic_reconfigure_options(
  cfg/CLIKDynamic.cfg
  cfg/CLIKPlannerDynamic.cfg
)

#######################################
## Declare ROS messages and services ##
#######################################

## Generate messages in the 'msg' folder
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate added messages and services with any dependencies listed here
# generate_messages(
#   DEPENDENCIES
#   std_msgs  # Or other packages containing msgs
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
 INCLUDE_DIRS include ${catkin_INCLUDE_DIRS}
 LIBRARIES constrained_ik constrained_ik_constraints moveit_clik_planner_plugin constrained_ik_plugin
 CATKIN_DEPENDS roscpp urdf eigen_conversions tf_conversions urdf moveit_ros_planning moveit_core dynamic_reconfigure kdl_parser cmake_modules industrial_collision_detection
 DEPENDS boost eigen orocos_kdl
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(include ${catkin_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS})

## Declare a cpp library
add_library(constrained_ik
            src/basic_kin.cpp
            src/constrained_ik.cpp
            src/constraint.cpp
            src/solver_state.cpp
            src/enum_types.cpp
            src/constrained_ik_utils.cpp
            src/constraint_group.cpp
)

target_link_libraries(constrained_ik ${catkin_LIBRARIES})

add_library(constrained_ik_constraints
            src/constraints/goal_orientation.cpp
            src/constraints/goal_position.cpp
            src/constraints/tool_position.cpp
            src/constraints/goal_tool_orientation.cpp
            src/constraints/goal_minimize_change.cpp
            src/constraints/goal_mid_joint.cpp
            src/constraints/goal_zero_jvel.cpp
            src/constraints/avoid_singularities.cpp
            src/constraints/joint_vel_limits.cpp
            src/constraints/avoid_obstacles.cpp
            src/constraints/avoid_joint_limits.cpp
            src/constraints/goal_tool_pointing.cpp
)
target_link_libraries(constrained_ik_constraints constrained_ik ${catkin_LIBRARIES})

add_dependencies(constrained_ik ${PROJECT_NAME}_gencfg)

add_library(constrained_ik_plugin
            src/moveit_interface/constrained_ik_plugin.cpp
)
target_link_libraries(constrained_ik_plugin constrained_ik constrained_ik_constraints ${catkin_LIBRARIES})

add_library(moveit_clik_planner_plugin
            src/moveit_interface/constrained_ik_planner_plugin.cpp
            src/moveit_interface/joint_interpolation_planner.cpp
            src/moveit_interface/cartesian_planner.cpp)

target_link_libraries(moveit_clik_planner_plugin constrained_ik constrained_ik_constraints ${catkin_LIBRARIES})

## Declare a cpp executable
# add_executable(constrained_ik_node src/constrained_ik_node.cpp)

## Add cmake target dependencies of the executable/library
## as an example, message headers may need to be generated before nodes
# add_dependencies(constrained_ik_node constrained_ik_generate_messages_cpp)

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/groovy/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS constrained_ik constrained_ik_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)

  add_rostest_gtest(test_basic_kin test/test_basic_kin.launch test/test_basic_kin.cpp )
  target_link_libraries(test_basic_kin constrained_ik constrained_ik_constraints)

  add_rostest_gtest(test_constrained_ik
    test/test_constrained_ik.launch
    test/test_constrained_ik.cpp )
  target_link_libraries(test_constrained_ik constrained_ik constrained_ik_constraints)
endif()

##############
## Examples ##
##############

## Add cpp example targets and link libraries
SET(EXAMPLE_LIBS
    constrained_ik
    constrained_ik_constraints
    ${catkin_LIBRARIES})

add_executable(constraints_from_yaml_example examples/constraints_from_yaml_example.cpp examples/example_utils.h)
target_link_libraries(constraints_from_yaml_example ${EXAMPLE_LIBS})

add_executable(avoid_joint_limits_example examples/avoid_joint_limits_example.cpp examples/example_utils.h)
target_link_libraries(avoid_joint_limits_example ${EXAMPLE_LIBS})

add_executable(avoid_obstacles_example examples/avoid_obstacles_example.cpp examples/example_utils.h)
target_link_libraries(avoid_obstacles_example ${EXAMPLE_LIBS})

add_executable(avoid_singularities_example examples/avoid_singularities_example.cpp examples/example_utils.h)
target_link_libraries(avoid_singularities_example ${EXAMPLE_LIBS})

add_executable(goal_mid_joint_example examples/goal_mid_joint_example.cpp examples/example_utils.h)
target_link_libraries(goal_mid_joint_example ${EXAMPLE_LIBS})

add_executable(goal_minimize_change_example examples/goal_minimize_change_example.cpp examples/example_utils.h)
target_link_libraries(goal_minimize_change_example ${EXAMPLE_LIBS})

add_executable(goal_orientation_example examples/goal_orientation_example.cpp examples/example_utils.h)
target_link_libraries(goal_orientation_example ${EXAMPLE_LIBS})

add_executable(goal_position_example examples/goal_position_example.cpp examples/example_utils.h)
target_link_libraries(goal_position_example ${EXAMPLE_LIBS})

add_executable(goal_tool_orientation_example examples/goal_tool_orientation_example.cpp examples/example_utils.h)
target_link_libraries(goal_tool_orientation_example ${EXAMPLE_LIBS})

add_executable(goal_tool_pointing_example examples/goal_tool_pointing_example.cpp examples/example_utils.h)
target_link_libraries(goal_tool_pointing_example ${EXAMPLE_LIBS})

add_executable(goal_zero_jvel_example examples/goal_zero_jvel_example.cpp examples/example_utils.h)
target_link_libraries(goal_zero_jvel_example ${EXAMPLE_LIBS})

add_executable(avoid_joint_vel_limits_example examples/avoid_joint_vel_limits_example.cpp examples/example_utils.h)
target_link_libraries(avoid_joint_vel_limits_example ${EXAMPLE_LIBS})

add_executable(goal_tool_position_example examples/goal_tool_position_example.cpp examples/example_utils.h)
target_link_libraries(goal_tool_position_example ${EXAMPLE_LIBS})


