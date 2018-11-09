cmake_minimum_required(VERSION 3.5)
project(camera_calibration_parsers)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)


# find dependencies
find_package(ament_cmake REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(console_bridge REQUIRED)
find_package(rclcpp REQUIRED)
# find_package(rclcpp_serialization REQUIRED)

# find_package(PythonLibs REQUIRED)
# find_package(PythonInterp REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem) # python)
# include_directories(include ${catkin_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${PYTHON_INCLUDE_DIRS})



find_package(PkgConfig)

find_package(python_cmake_module REQUIRED)
_ament_cmake_python_register_environment_hook()


if (ANDROID)
    find_package(yaml-cpp)
    add_definitions(-DHAVE_NEW_YAMLCPP)
else()
    find_package(yaml-cpp REQUIRED)
    pkg_check_modules(YAML_CPP yaml-cpp)
    add_definitions(-DHAVE_NEW_YAMLCPP)
    link_directories(${YAML_CPP_LIBRARY_DIRS})
endif()
# include_directories(${YAML_CPP_INCLUDE_DIRS})

# define the library
add_library(${PROJECT_NAME} SHARED
  src/parse.cpp
  src/parse_ini.cpp
  src/parse_yml.cpp
)

#add_library(${PROJECT_NAME}_wrapper
#  src/parse_wrapper.cpp)



ament_target_dependencies(${PROJECT_NAME}
    "rclcpp"
    "rcutils"
    "builtin_interfaces"
    "sensor_msgs"
    "console_bridge")

target_link_libraries(${PROJECT_NAME} yaml-cpp ${Boost_LIBRARIES})
#ament_target_dependencies(${PROJECT_NAME}_wrapper
#    "rclcpp"
#    "rcutils"
#    "builtin_interfaces"
#    "sensor_msgs"
#    "console_bridge")

target_include_directories(${PROJECT_NAME} PUBLIC include ${YAML_CPP_INCLUDE_DIR})
#target_include_directories(${PROJECT_NAME}_wrapper PUBLIC include ${PYTHON_INCLUDE_DIRS})
    
# ament_target_dependencies(${PROJECT_NAME} "sensor_msgs" "console_bridge" "rclcpp" ${YAML_CPP_LIBRARIES} ${Boost_FILESYSTEM_LIBRARY})
# ament_target_dependencies(${PROJECT_NAME}_wrapper ${PROJECT_NAME} ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${PYTHON_LIBRARIES})

# Don't prepend wrapper library name with lib and add to Python libs.
#set_target_properties(${PROJECT_NAME}_wrapper PROPERTIES
#        PREFIX ""
#        LIBRARY_OUTPUT_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_PYTHON_DESTINATION}
#        )

# add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})


install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)


install(
  DIRECTORY "include/"
  DESTINATION include
)

# define the exe to convert
add_executable(convert src/convert.cpp)
target_include_directories(convert PUBLIC include)

ament_target_dependencies(convert ${PROJECT_NAME} 
"rclcpp"
"rcutils"
"builtin_interfaces"
"sensor_msgs"
"console_bridge")

target_link_libraries(convert ${PROJECT_NAME})

install(
  TARGETS convert
  DESTINATION bin
)

#install(TARGETS ${PROJECT_NAME}_wrapper
#  DESTINATION "${PYTHON_INSTALL_DIR}/${PROJECT_NAME}"
#)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

ament_package()


if(CATKIN_ENABLE_TESTING)
  add_subdirectory(test)
endif()
