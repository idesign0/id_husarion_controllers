cmake_minimum_required(VERSION 3.11)
project(low_pass_filter)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(PACKAGE_DEPENDENCIES
    ament_cmake
    control_toolbox
    controller_interface
    generate_parameter_library
    hardware_interface
    pluginlib
    rclcpp
    rclcpp_lifecycle
    realtime_tools)

foreach(PACKAGE IN ITEMS ${PACKAGE_DEPENDENCIES})
  find_package(${PACKAGE} REQUIRED)
endforeach()

generate_parameter_library(low_pass_filter_parameters
                           src/low_pass_filter_parameters.yaml)

add_library(low_pass_filter SHARED src/low_pass_filter.cpp)
# The upstream ros2_control `control_toolbox` package also builds a library named
# `low_pass_filter` (its control_filters/low_pass_filter). In a merge-install
# workspace both land in the same lib/ dir with the same file name. On Linux one
# silently clobbers the other; on macOS the linker refuses ("can't link a dylib
# with itself" — same install_name) because this package links control_toolbox.
# Give this package's library a distinct file name to avoid the collision.
set_target_properties(low_pass_filter PROPERTIES OUTPUT_NAME husarion_low_pass_filter)

target_include_directories(
  low_pass_filter PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                         $<INSTALL_INTERFACE:include/low_pass_filter>)
target_link_libraries(low_pass_filter PUBLIC low_pass_filter_parameters)
ament_target_dependencies(low_pass_filter PUBLIC ${PACKAGE_DEPENDENCIES})
# Causes the visibility macros to use dllexport rather than dllimport, which is
# appropriate when building the dll but not consuming it.
target_compile_definitions(low_pass_filter
                           PRIVATE "low_pass_filter_BUILDING_DLL")
pluginlib_export_plugin_description_file(controller_interface
                                         low_pass_filter_plugin.xml)

install(DIRECTORY include/ DESTINATION include/low_pass_filter)

install(
  TARGETS ${PROJECT_NAME} low_pass_filter_parameters
  EXPORT export_${PROJECT_NAME}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()
