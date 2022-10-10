# Copyright(c) 2020 livoxtech limited.

cmake_minimum_required(VERSION 3.22)
project(livox_ros2_driver)

# Default to C99
if (NOT CMAKE_C_STANDARD)
    set(CMAKE_C_STANDARD 99)
endif ()

# Default to C++14
if (NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
endif ()

if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif ()

# Printf version info
include(cmake/version.cmake)
project(${PROJECT_NAME} VERSION ${LIVOX_ROS_DRIVER_VERSION} LANGUAGES CXX)

find_package(PCL REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(livox_interfaces REQUIRED)
find_package(livox_sdk_vendor REQUIRED) # must appear before livox_sdk
find_package(livox_sdk REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(pcl_msgs REQUIRED)
find_package(rclcpp REQUIRED)

find_package(PkgConfig)
pkg_check_modules(APR apr-1)
if (APR_FOUND)
    message(${APR_INCLUDE_DIRS})
    message(${APR_LIBRARIES})
endif (APR_FOUND)

link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

add_executable(${PROJECT_NAME}
        livox_ros2_driver/lvx_file.cpp
        livox_ros2_driver/ldq.cpp
        livox_ros2_driver/lds.cpp
        livox_ros2_driver/lds_lvx.cpp
        livox_ros2_driver/lds_lidar.cpp
        livox_ros2_driver/lds_hub.cpp
        livox_ros2_driver/lddc.cpp
        livox_ros2_driver/livox_ros2_driver.cpp
        timesync/timesync.cpp
        timesync/user_uart/user_uart.cpp
        common/comm/comm_protocol.cpp
        common/comm/sdk_protocol.cpp
        common/comm/gps_protocol.cpp
        )

target_compile_options(${PROJECT_NAME}
        PRIVATE $<$<CXX_COMPILER_ID:GNU>:-Wno-missing-field-initializers -fPIC>
        )

target_include_directories(${PROJECT_NAME}
        SYSTEM PRIVATE
        ${APR_INCLUDE_DIRS}
        ${PCL_INCLUDE_DIRS}
        ${geometry_msgs_INCLUDE_DIRS}
        ${livox_interfaces_INCLUDE_DIRS}
        ${livox_sdk_INCLUDE_DIRS}
        ${pcl_conversions_INCLUDE_DIRS}
        common
        common/comm
        common/rapdidxml
        common/rapidjson
        livox_ros2_driver
        timesync
        timesync/user_uart
        )

target_link_libraries(${PROJECT_NAME}
        ${APR_LIBRARIES}
        ${PCL_LIBRARIES}
        ${geometry_msgs_LIBRARIES}
        ${livox_interfaces_LIBRARIES}
        ${livox_sdk_LIBRARIES}
        ${pcl_conversions_LIBRARIES}
        rclcpp::rclcpp
        )

install(TARGETS ${PROJECT_NAME}
        DESTINATION lib/${PROJECT_NAME})

ament_package()
