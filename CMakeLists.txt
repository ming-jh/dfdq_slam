cmake_minimum_required(VERSION 2.8.3)
project(dfdq_slam)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++17")
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -Wall -g")

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  nav_msgs
  sensor_msgs
  roscpp
  rospy
  rosbag
  std_msgs
  tf
  eigen_conversions
)
find_package(Boost REQUIRED COMPONENTS timer thread)

find_package(Eigen3)
if(NOT EIGEN3_FOUND)
  # Fallback to cmake_modules
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  set(EIGEN3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS})
  set(EIGEN3_LIBRARIES ${EIGEN_LIBRARIES})  # Not strictly necessary as Eigen is head only
  # Possibly map additional variables to the EIGEN3_ prefix.
else()
  set(EIGEN3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
endif()
set(PCL_DIR "/home/robot/lib/pcl-pcl-1.8.1/build")
find_package(PCL REQUIRED)
find_package(Ceres REQUIRED)
find_package(GTSAM REQUIRED)
include_directories(
  include
  ${CERES_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS} 
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
  ${GTSAM_INCLUDE_DIR}
)

link_directories(
  include
  ${PCL_LIBRARY_DIRS}
  ${CERES_LIBRARY_DIRS}
  ${GTSAM_LIBRARY_DIRS}
)


catkin_package(
  CATKIN_DEPENDS geometry_msgs nav_msgs roscpp rospy std_msgs
  DEPENDS EIGEN3 PCL  
  INCLUDE_DIRS include
)


add_executable(dfdq_slam_laser_processing_node src/laserProcessingNode.cpp src/laserProcessingClass.cpp src/lidar.cpp)
target_link_libraries(dfdq_slam_laser_processing_node ${EIGEN3_LIBRARIES} ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${CERES_LIBRARIES})

add_executable(dfdq_slam_odom_estimation_localization_node src/odomEstimationLocalizationNode.cpp src/lidarOptimization.cpp src/lidar.cpp src/odomEstimationLocalizationClass.cpp)
target_link_libraries(dfdq_slam_odom_estimation_localization_node ${EIGEN3_LIBRARIES} ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${CERES_LIBRARIES})

add_executable(dfdq_slam_odom_estimation_mapping_node src/odomEstimationMappingNode.cpp src/lidarOptimization.cpp src/lidar.cpp src/odomEstimationMappingClass.cpp)
target_link_libraries(dfdq_slam_odom_estimation_mapping_node ${EIGEN3_LIBRARIES} ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${CERES_LIBRARIES})

add_executable(dfdq_slam_map_optimization_node src/mapOptimizationNode.cpp src/mapOptimizationClass.cpp src/lidar.cpp src/lidarOptimization.cpp)
target_link_libraries(dfdq_slam_map_optimization_node ${EIGEN3_LIBRARIES} ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${CERES_LIBRARIES} gtsam "/usr/lib/x86_64-linux-gnu/libtbb.so.2")