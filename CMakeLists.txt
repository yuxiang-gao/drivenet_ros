cmake_minimum_required(VERSION 3.1)
project(drivenet)

set (CMAKE_CXX_STANDARD 11)
set(Driveworks_INCLUDE_DIR /usr/local/driveworks/include)
set(Driveworks_LIBRARY /usr/local/driveworks/lib)
set(GLFW_INCLUDE_DIR /usr/share/visionworks/VisionWorks-1.6-Samples/3rdparty/glfw3/include)
set(GLFW_LIBRARY /usr/share/visionworks/VisionWorks-1.6-Samples/3rdparty/glfw3/libs)
# FindDriveworks.cmake, ArchConfiguration.cmake, and LibFindMacros.cmake were needed for my setup they are taken from driveworks/samples/cmake/
# ArchConfiguration.cmake was the only file that needed small changes, remove the fatal error on line 17 and add the following lines in its place
         set(VIBRANTE TRUE)
         add_definitions(-DVIBRANTE)
# this is the path I placed the driveworks cmake files in
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

#add_subdirectory(include/driveworks_common)
#add_subdirectory(include/drivenet/common)

find_package(CUDA REQUIRED)
find_package(Threads REQUIRED)
find_package(GLEW REQUIRED)
find_package(Driveworks REQUIRED)
find_package(OpenCV REQUIRED)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  geometry_msgs
  cv_bridge 
  image_transport
)

catkin_package(
  LIBRARIES ${PROJECT_NAME}
)
link_directories(
  ${Driveworks_LIBRARY} ${GLFW_LIBRARY})

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${Driveworks_INCLUDE_DIR}
  ${CUDA_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${GLEW_INCLUDE_DIR}
  ${GLFW_INCLUDE_DIR}
)

# TODO: add a FindNvmedia.cmake file for this? Why does it not exist?
include_directories(/usr/share/visionworks/sources/3rdparty/nvmedia/
#/usr/share/visionworks/VisionWorks-1.6-Samples/3rdparty/glfw3/include
/usr/local/driveworks/include
/usr/local/cuda/include
)

set(LIBRARIES
    ${Driveworks_LIBRARIES}
    ${GLFW_LIBRARIES}
    ${OpenCV_LIBS} 
    ${CUDA_LIBRARY}
    ${catkin_LIBRARIES}
    #dw_samples_common
    nvmedia
    driveworks
    GLESv2
    EGL
    drm
)

# ros node name template
set(NODE_NAME ${PROJECT_NAME}_node)

set(COMMON_SOURCE 
${CMAKE_CURRENT_SOURCE_DIR}/include/driveworks_common/SampleFramework.cpp
${CMAKE_CURRENT_SOURCE_DIR}/include/driveworks_common/ConsoleColor.cpp
${CMAKE_CURRENT_SOURCE_DIR}/include/driveworks_common/ProgramArguments.cpp)

cuda_add_executable(${NODE_NAME} src/main.cpp 
    ${CMAKE_CURRENT_SOURCE_DIR}/include/drivenet/common/DriveNet.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/drivenet/common/common.cpp
    src/cv_connection.cpp
    ${COMMON_SOURCE}
)
add_dependencies(${NODE_NAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(${NODE_NAME} PRIVATE ${LIBRARIES})

install(TARGETS ${NODE_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
