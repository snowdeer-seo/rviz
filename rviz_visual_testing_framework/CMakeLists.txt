cmake_minimum_required(VERSION 3.5)

project(rviz_visual_testing_framework)

# Options to decide whether or not the visual test will run.
option(EnableVisualTests "decides whether or not enable the tests")

# Path variables needed to save and find screenshots for image comparison.
add_definitions(-D_BUILD_DIR_PATH="${CMAKE_CURRENT_BINARY_DIR}")
add_definitions(-D_SRC_DIR_PATH="${CMAKE_CURRENT_SOURCE_DIR}")

# Creates a directory where the test and reference screenshots will be saved.
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/test_images)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/test/reference_images)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)

# Qt5 boilerplate options from http://doc.qt.io/qt-5/cmake-manual.html
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

find_package(rviz_common REQUIRED)

find_package(rviz_ogre_vendor REQUIRED)

find_package(Qt5 REQUIRED COMPONENTS Widgets Test)

find_package(ament_cmake_gtest REQUIRED)
ament_find_gtest()

set(visual_test_framework_source_files
  test/example_nodes.hpp
  include/rviz_visual_testing_framework/page_objects/base_page_object.hpp
  src/page_objects/base_page_object.cpp
  include/rviz_visual_testing_framework/page_objects/camera_display_page_object.hpp
  src/page_objects/camera_display_page_object.cpp
  include/rviz_visual_testing_framework/page_objects/grid_display_page_object.hpp
  src/page_objects/grid_display_page_object.cpp
  include/rviz_visual_testing_framework/page_objects/image_display_page_object.hpp
  src/page_objects/image_display_page_object.cpp
  include/rviz_visual_testing_framework/page_objects/page_object_with_window.hpp
  src/page_objects/page_object_with_window.cpp
  include/rviz_visual_testing_framework/page_objects/point_cloud_display_page_object.hpp
  src/page_objects/point_cloud_display_page_object.cpp
  include/rviz_visual_testing_framework/page_objects/polygon_display_page_object.hpp
  src/page_objects/polygon_display_page_object.cpp
  src/internal/rviz_scene_test.cpp
  include/rviz_visual_testing_framework/internal/rviz_scene_test.hpp
  include/rviz_visual_testing_framework/internal/display_handler.hpp
  src/internal/display_handler.cpp
  src/internal/image_tester.cpp
  include/rviz_visual_testing_framework/internal/image_tester.hpp
  include/rviz_visual_testing_framework/internal/test_helpers.hpp
  src/internal/test_helpers.cpp
  include/rviz_visual_testing_framework/internal/executor.hpp
  src/internal/executor.cpp
  include/rviz_visual_testing_framework/internal/visual_test.hpp
  src/internal/visual_test.cpp
  include/rviz_visual_testing_framework/visual_test_fixture.hpp
  src/visual_test_fixture.cpp)

set(visual_tests_target_libaries
  rviz_common::rviz_common
  Qt5::Widgets
  Qt5::Test)

# TODO(wjwwood): this block is to setup the windeployqt tool, could be removed later.
if(Qt5_FOUND AND WIN32 AND TARGET Qt5::qmake AND NOT TARGET Qt5::windeployqt)
  get_target_property(_qt5_qmake_location Qt5::qmake IMPORTED_LOCATION)

  execute_process(
    COMMAND "${_qt5_qmake_location}" -query QT_INSTALL_PREFIX
    RESULT_VARIABLE return_code
    OUTPUT_VARIABLE qt5_install_prefix
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
endif()

add_library(rviz_visual_testing_framework STATIC ${visual_test_framework_source_files})

target_include_directories(rviz_visual_testing_framework
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    ${GTEST_INCLUDE_DIRS})

target_link_libraries(rviz_visual_testing_framework
  ${visual_tests_target_libaries})

# export information to downstream packages
ament_export_interfaces(rviz_visual_testing_framework)
ament_export_dependencies(Qt5)
ament_export_dependencies(rviz_common)



install(
  TARGETS rviz_visual_testing_framework
  EXPORT rviz_visual_testing_framework
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(
  DIRECTORY include/
  DESTINATION include
)

if(BUILD_TESTING)
  # TODO(wjwwood): replace this with ament_lint_auto() and/or add the copyright linter back
  find_package(ament_cmake_cppcheck REQUIRED)
  find_package(ament_cmake_cpplint REQUIRED)
  find_package(ament_cmake_lint_cmake REQUIRED)
  find_package(ament_cmake_uncrustify REQUIRED)
  ament_cppcheck()
  ament_cpplint()
  ament_lint_cmake()
  ament_uncrustify()

  if(EnableVisualTests STREQUAL "TRUE")
    ament_add_gtest(
      example_test
      test/example_test.cpp)
    if(TARGET example_test)
      target_link_libraries(
        example_test
        rviz_visual_testing_framework)
      if(EXISTS ${qt5_install_prefix})
        add_custom_command(TARGET example_test
          POST_BUILD
          COMMAND
          bash -c "${qt5_install_prefix}/bin/windeployqt.exe ${CMAKE_CURRENT_BINARY_DIR}/Release")
      endif()
    endif()
  endif()
endif()

ament_package()
