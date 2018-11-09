cmake_minimum_required(VERSION 3.5)
project(camera_info_manager)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake_python REQUIRED)
find_package(ament_cmake_ros REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rcutils)
find_package(builtin_interfaces REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(camera_calibration_parsers REQUIRED)
find_package(image_transport REQUIRED)

find_package(Boost)

include_directories(${catkin_INCLUDE_DIRS})
include_directories(include)

# add a library
add_library(${PROJECT_NAME} src/camera_info_manager.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})

ament_target_dependencies(${PROJECT_NAME}
  "rclcpp"
  "rcutils"
  "builtin_interfaces"
  "sensor_msgs"
  "camera_calibration_parsers"
  "image_transport")

install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
)
install(DIRECTORY include/
        DESTINATION include
)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

ament_package()


if(CATKIN_ENABLE_TESTING)
  find_package(rostest)
  # Unit test uses gtest, but needs rostest to create a ROS environment.
  # Hence, it must be created as a normal executable, not using
  # catkin_add_gtest() which runs an additional test without rostest.
  add_executable(unit_test tests/unit_test.cpp)
  target_link_libraries(unit_test ${PROJECT_NAME} ${GTEST_LIBRARIES} ${catkin_LIBRARIES})

  add_rostest(tests/unit_test.test DEPENDENCIES unit_test)
endif()
