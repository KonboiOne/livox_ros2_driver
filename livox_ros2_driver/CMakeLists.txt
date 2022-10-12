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

include(ExternalProject)
set(livox_sdk_INSTALL_PATH ${CMAKE_CURRENT_BINARY_DIR}/livox-sdk)
set(livox_sdk_LIBRARY ${livox_sdk_INSTALL_PATH}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}livox_sdk_static${CMAKE_STATIC_LIBRARY_SUFFIX})
ExternalProject_Add(livox-sdk
        GIT_REPOSITORY https://github.com/KonboiOne/Livox-SDK.git
        GIT_TAG bb64378d
        GIT_SHALLOW OFF
        DOWNLOAD_NO_PROGRESS ON
        GIT_PROGRESS OFF
        LOG_UPDATE ON
        LOG_DOWNLOAD ON
        LOG_CONFIGURE ON
        LOG_INSTALL ON
        LOG_BUILD ON
        LOG_OUTPUT_ON_FAILURE ON
        TIMEOUT 60
        CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=Release
        -DCMAKE_INSTALL_PREFIX=${livox_sdk_INSTALL_PATH}
        BUILD_BYPRODUCTS ${livox_sdk_LIBRARY}
        )

find_package(PCL REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(livox_interfaces REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(pcl_msgs REQUIRED)
find_package(rclcpp REQUIRED)

link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

add_executable(${PROJECT_NAME})
add_dependencies(${PROJECT_NAME} livox-sdk)
target_sources(${PROJECT_NAME}
        PRIVATE
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
        ${PCL_INCLUDE_DIRS}
        ${geometry_msgs_INCLUDE_DIRS}
        ${livox_interfaces_INCLUDE_DIRS}
        ${livox_sdk_INSTALL_PATH}/include
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
        ${PCL_LIBRARIES}
        ${geometry_msgs_LIBRARIES}
        ${livox_interfaces_LIBRARIES}
        ${livox_sdk_LIBRARY}
        ${pcl_conversions_LIBRARIES}
        rclcpp::rclcpp
        )

install(TARGETS ${PROJECT_NAME}
        DESTINATION lib/${PROJECT_NAME})

ament_package()
