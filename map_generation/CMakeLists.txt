cmake_minimum_required(VERSION 2.8.3)
project(map_generation)


find_package(catkin REQUIRED COMPONENTS
  roscpp message_generation  eigen_conversions  geometry_msgs tf moveit_ros_planning_interface  moveit_core)

find_package(octomap REQUIRED)
find_package(HDF5 REQUIRED)




add_message_files(
   FILES
   WsSphere.msg
   WorkSpace.msg
 )


generate_messages(
   DEPENDENCIES
   std_msgs geometry_msgs # Or other packages containing msgs
 )


catkin_package(
  INCLUDE_DIRS
      include
  LIBRARIES
      discretization
      dataset
      utility
      reachability
      map_generation
  CATKIN_DEPENDS message_runtime
#  DEPENDS system_lib
)


include_directories(
  include ${catkin_INCLUDE_DIRS} ${OCTOMAP_INCLUDE_DIRS} ${HDF5_INCLUDE_DIR}
)


 add_library(discretization   src/${PROJECT_NAME}/discretization.cpp )
 add_library(dataset src/${PROJECT_NAME}/hdf5_dataset.cpp)
 add_library(utility  src/${PROJECT_NAME}/utility.cpp)
 add_library(reachability  src/${PROJECT_NAME}/reachability.cpp)
 add_library(map_generation  src/${PROJECT_NAME}/map_generation.cpp)

 target_link_libraries(discretization  ${catkin_LIBRARIES} ${OCTOMAP_LIBRARIES})
 target_link_libraries(utility  ${catkin_LIBRARIES})
 target_link_libraries(reachability  ${catkin_LIBRARIES} utility)
 target_link_libraries(dataset  ${catkin_LIBRARIES} -lhdf5 -lhdf5_cpp utility)
 target_link_libraries(map_generation  ${catkin_LIBRARIES} discretization utility reachability dataset)


 install(TARGETS discretization dataset utility reachability map_generation
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
 )



add_executable(map_generation_node src/nodes/map_generation_node.cpp)
add_dependencies(map_generation_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(map_generation_node map_generation ${catkin_LIBRARIES})
