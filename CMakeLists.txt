cmake_minimum_required(VERSION 2.8.3)
project(orb_slam_ros)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  cv_bridge
  image_transport
  sensor_msgs
  pcl_conversions
  pcl_ros
)

#FIND_PACKAGE( PCL REQUIRED COMPONENTS common io visualization filters )
FIND_PACKAGE( PCL REQUIRED)

# PCL
ADD_DEFINITIONS( ${PCL_DEFINITIONS} )
INCLUDE_DIRECTORIES( ${PCL_INCLUDE_DIRS}  )
LINK_LIBRARIES( ${PCL_LIBRARY_DIRS} )

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

# LIST(APPEND CMAKE_MODULE_PATH ${CMAKE_INSTALL_PREFIX}/lib/cmake/g2o)
# FIND_PACKAGE(G2O REQUIRED)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES aadet
#  CATKIN_DEPENDS roscpp rospy std_msgs
#  DEPENDS system_lib
)

include_directories(
  ${catkin_INCLUDE_DIRS}
)

# link_directories(${CATKIN_PACKAGE_LIB_DESTINATION})

include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

# set(GO2_LIBS ${CMAKE_INSTALL_PREFIX}/lib/libg2o_cli.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_core.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_csparse_extension.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_ext_csparse.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_ext_freeglut_minimal.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_hierarchical.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_interface.so
# ${CMAKE_INSTALL_PREFIX}/lib/libg2o_opengl_helper.so
# libg2o_parser.so
# libg2o_simulator.so
# libg2o_solver_csparse.so
# libg2o_solver_dense.so
# libg2o_solver_eigen.so
# libg2o_solver_pcg.so
# libg2o_solver_slam2d_linear.so
# libg2o_solver_structure_only.so
# libg2o_stuff.so
# libg2o_types_data.so
# libg2o_types_icp.so
# libg2o_types_sba.so
# libg2o_types_sclam2d.so
# libg2o_types_sim3.so
# libg2o_types_slam2d.so
# libg2o_types_slam2d_addons.so
# libg2o_types_slam3d.so
# libg2o_types_slam3d_addons.so
# )

# message(FATAL_ERROR "Files are ${G2O_CORE_LIBRARY}" )

# Stereo
add_executable(stereo_node_KITTI 
src/ros_stereo.cc
src/System.cc
src/Tracking.cc
src/LocalMapping.cc
src/LoopClosing.cc
src/ORBextractor.cc
src/ORBmatcher.cc
src/FrameDrawer.cc
src/Converter.cc
src/MapPoint.cc
src/KeyFrame.cc
src/Map.cc
src/MapDrawer.cc
src/Optimizer.cc
src/PnPsolver.cc
src/Frame.cc
src/KeyFrameDatabase.cc
src/Sim3Solver.cc
src/Initializer.cc
src/Viewer.cc
src/SlamDataPub.cc
)
target_link_libraries(stereo_node_KITTI 
${catkin_LIBRARIES}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
# ${G2O_CORE_LIBRARY}
# ${G2O_STUFF_LIBRARY}
${CMAKE_INSTALL_PREFIX}/lib/libDBoW2.so
${CMAKE_INSTALL_PREFIX}/lib/libg2o.so
# /home/docker/visual_odometry_tests/dev/install/lib/libg2o_core.so
# ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
# ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
${PCL_LIBRARIES}
)


# Mono
add_executable(mono_node_KITTI 
src/ros_mono.cc
src/System.cc
src/Tracking.cc
src/LocalMapping.cc
src/LoopClosing.cc
src/ORBextractor.cc
src/ORBmatcher.cc
src/FrameDrawer.cc
src/Converter.cc
src/MapPoint.cc
src/KeyFrame.cc
src/Map.cc
src/MapDrawer.cc
src/Optimizer.cc
src/PnPsolver.cc
src/Frame.cc
src/KeyFrameDatabase.cc
src/Sim3Solver.cc
src/Initializer.cc
src/Viewer.cc
src/SlamDataPub.cc
)
target_link_libraries(mono_node_KITTI 
${catkin_LIBRARIES}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${G2O_CORE_LIBRARY}
${G2O_STUFF_LIBRARY}
${CMAKE_INSTALL_PREFIX}/lib/libDBoW2.so
${CMAKE_INSTALL_PREFIX}/lib/libg2o.so
${helloworld_SRC}
${PCL_LIBRARIES}
)


install(TARGETS mono_node_KITTI stereo_node_KITTI
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)






