cmake_minimum_required(VERSION 2.8)
project(ORB_SLAM2)

#IF(NOT CMAKE_BUILD_TYPE)
#  SET(CMAKE_BUILD_TYPE Release)
#ENDIF()

#MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

if(WIN32)
#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  /Wall  /O2 -march=native ")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Wall  /O2 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Wall /O2 /Zi")
else()
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")
endif()

# Check C++11 or C++0x support
#include(CheckCXXCompilerFlag)
#CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
#CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
#if(COMPILER_SUPPORTS_CXX11)
#   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
#   add_definitions(-DCOMPILEDWITHC11)
#   message(STATUS "Using flag -std=c++11.")
#elseif(COMPILER_SUPPORTS_CXX0X)
#   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
#   add_definitions(-DCOMPILEDWITHC0X)
#   message(STATUS "Using flag -std=c++0x.")
#else()
#   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
#endif()

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



#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

include_directories(
${EIGEN3_INCLUDE_DIR}
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${Pangolin_INCLUDE_DIRS}
)

set(ORBSLAM_DEBUG_POSTFIX "-d")
set(ORBSLAM_LINK_FLAG "/DEBUG")

add_subdirectory(Thirdparty/DBoW2)
add_subdirectory(Thirdparty/g2o)

add_definitions(-DDBOW2_SHARED_LIBS)
add_definitions(-DG2O_SHARED_LIBS)
add_definitions(-DORB_SLAM2_SHARED_LIBS)

add_library(${PROJECT_NAME} SHARED
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
)

if(WIN32)

set(LIB_DBoW2 optimized ${PROJECT_BINARY_DIR}/Release/DBoW2.lib debug ${PROJECT_BINARY_DIR}/Debug/DBoW2-d.lib)
set(LIB_g2o optimized ${PROJECT_BINARY_DIR}/Release/g2o.lib debug ${PROJECT_BINARY_DIR}/Debug/g2o-d.lib)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${LIB_DBoW2}
${LIB_g2o}
)

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

else()
target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
)
endif()


# Build examples

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)

add_definitions(-DCOMPILEDWITHC11)

add_executable(rgbd_tum
Examples/RGB-D/rgbd_tum.cc)
target_link_libraries(rgbd_tum ${PROJECT_NAME})
set_target_properties(rgbd_tum PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)

add_executable(stereo_kitti
Examples/Stereo/stereo_kitti.cc)
target_link_libraries(stereo_kitti ${PROJECT_NAME})
set_target_properties(stereo_kitti PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

add_executable(stereo_euroc
Examples/Stereo/stereo_euroc.cc)
target_link_libraries(stereo_euroc ${PROJECT_NAME})
set_target_properties(stereo_euroc PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})


set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)

add_executable(mono_tum
Examples/Monocular/mono_tum.cc)
target_link_libraries(mono_tum ${PROJECT_NAME})
set_target_properties(mono_tum PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

add_executable(mono_kitti
Examples/Monocular/mono_kitti.cc)
target_link_libraries(mono_kitti ${PROJECT_NAME})
set_target_properties(mono_kitti PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

add_executable(mono_euroc
Examples/Monocular/mono_euroc.cc)
target_link_libraries(mono_euroc ${PROJECT_NAME})
set_target_properties(mono_euroc PROPERTIES DEBUG_POSTFIX ${ORBSLAM_DEBUG_POSTFIX}
						LINK_FLAGS ${ORBSLAM_LINK_FLAG})

