cmake_minimum_required(VERSION 3.5)
project(franka_hardware)

# Default to C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Default to release
set(default_build_type "Release")
# Respect the build type from user and ide
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic -Werror=unused-variable)
endif()

option(CHECK_TIDY "Adds clang-tidy tests" OFF)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
find_package(std_srvs REQUIRED)
find_package(franka_msgs REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(Franka REQUIRED) # Franka_DIR will provide the path to the libfranka installation
find_package(Boost COMPONENTS thread chrono REQUIRED)

# Mujoco dependencies
# set(OpenGL_GL_PREFERENCE GLVND)
# find_package(glfw3 3.3 REQUIRED)     # GLFW3 here,
# find_package(mujoco REQUIRED)
find_package(yaml-cpp REQUIRED)
find_package(Eigen3 REQUIRED)

###### Real robot ###############
add_library(franka_hardware
        SHARED
        src/real/franka_multi_hardware_interface.cpp
        src/real/robot.cpp
        src/real/franka_param_service_server.cpp
        src/real/franka_error_recovery_service_server.cpp
        src/common/franka_executor.cpp
        src/common/control_mode.cpp
        src/common/helper_functions.cpp
)

target_include_directories(
        franka_hardware
        PRIVATE
        include
        ${EIGEN3_INCLUDE_DIRS}
        ${YAML_CPP_INCLUDE_DIRS}
)

ament_target_dependencies(
        franka_hardware
        hardware_interface
        Franka
        pluginlib
        rclcpp
        rclcpp_action
        rclcpp_components
        franka_msgs
        std_srvs
)

target_link_libraries(
        franka_hardware
        Boost::thread
        Boost::chrono
        ${YAML_CPP_LIBRARIES}
)

pluginlib_export_plugin_description_file(hardware_interface franka_hardware.xml)

# Install scripts
install(
        TARGETS franka_hardware
        DESTINATION lib
)
install(
        DIRECTORY include/
        DESTINATION include
)

if(BUILD_TESTING)
  
endif()

ament_export_include_directories(
        include
)
ament_export_libraries(
        franka_hardware
)
ament_export_dependencies(
        hardware_interface
        pluginlib
        rclcpp
)

ament_package()
