cmake_minimum_required(VERSION 2.8)
project(image_geometry)

find_package(catkin REQUIRED sensor_msgs)

set(ENABLE_CUDA OFF)

if(ENABLE_CUDA)
  add_definitions(-DGPU_EN)
  #   TX1 : OpenCV 2.4(OpenCV4Tegra is based on OpenCV2.4)
  #   Ubuntu 14/16: OpenCV 3.1
  exec_program(uname ARGS -p OUTPUT_VARIABLE CMAKE_SYSTEM_NAME2)
  if ( CMAKE_SYSTEM_NAME2 MATCHES "aarch64" ) # Jetson TX1
    SET(OCV_VERSION "2.4")
    add_definitions(-DTX1)
  else() # Ubuntu Desktop
    SET(OCV_VERSION "3.1")
    add_definitions(-Dx86)
  endif()
  find_package(OpenCV ${OCV_VERSION} REQUIRED)
else()
  find_package(OpenCV REQUIRED)
endif()

catkin_package(CATKIN_DEPENDS sensor_msgs
               DEPENDS OpenCV
               INCLUDE_DIRS include
               LIBRARIES ${PROJECT_NAME}
)

catkin_python_setup()

include_directories(${catkin_INCLUDE_DIRS}
                           ${OpenCV_INCLUDE_DIRS}
)
include_directories(include)

# add a library
add_library(${PROJECT_NAME} src/pinhole_camera_model.cpp src/stereo_camera_model.cpp)
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})
add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/
)

install(TARGETS ${PROJECT_NAME}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

# add tests
if(CATKIN_ENABLE_TESTING)
  add_subdirectory(test)
endif()
