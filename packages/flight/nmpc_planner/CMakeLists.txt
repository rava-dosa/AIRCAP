cmake_minimum_required(VERSION 2.8.3)
project(nmpc_planner)


find_package(cmake_modules REQUIRED)
SET (CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../CMakeModules)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  nav_msgs
  sensor_msgs
  roscpp
  rospy
  std_msgs
  uav_msgs
  tf
  mav_msgs #This is for the firefly utility in gazebo, not telekyb
  generic_potential_field
  dynamic_reconfigure
)


find_package(Boost QUIET REQUIRED COMPONENTS system thread)
#find_package(IPOPT REQUIRED)
find_package(Eigen REQUIRED)


include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

generate_dynamic_reconfigure_options(
  cfg/nmpcPlannerParams.cfg
)

catkin_package(
#  INCLUDE_DIRS include
#  CATKIN_DEPENDS roscpp rospy
CATKIN_DEPENDS roscpp rospy
LIBRARIES nmpc_solver
)





INCLUDE_DIRECTORIES(${catkin_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${generic_potential_field_INCLUDE_DIRS})

# INCLUDE_DIRECTORIES(${catkin_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS}  ${IPOPT_INCLUDE_DIRS} ${generic_potential_field_INCLUDE_DIRS})

add_library(nmpc_solver
	include/solver/MPCsolver.cpp
	include/solver/ldl.c 
	include/solver/util.c 
	include/solver/matrix_support.c  
	include/solver/solver.c
)

# add_library(nmpc_ipopt
# 	src/nmpc_ipopt.cpp
# )

add_dependencies(nmpc_solver
        uav_msgs_generate_messages_cpp
)

target_link_libraries(nmpc_solver
        ${Boost_LIBRARIES}
        ${catkin_LIBRARIES}
        ${generic_potential_field_LIBRARIES}
)

# target_link_libraries(nmpc_ipopt
#         ${Boost_LIBRARIES}
#         ${catkin_LIBRARIES}
#         ${IPOPT_LIBRARIES}
# )


add_executable(nmpc_planner src/nmpc_planner.cpp)
add_dependencies(nmpc_planner ${PROJECT_NAME}_gencfg)
target_link_libraries(nmpc_planner ${catkin_LIBRARIES} nmpc_solver)
# target_link_libraries(nmpc_planner ${catkin_LIBRARIES} nmpc_solver nmpc_ipopt)

