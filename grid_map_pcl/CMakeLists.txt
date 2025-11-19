cmake_minimum_required(VERSION 3.5.1)
project(grid_map_pcl)

set(CMAKE_CXX_STANDARD 17)
add_compile_options(-Wall -Wextra -Wpedantic)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(SRC_FILES
  src/GridMapPclConverter.cpp
  src/GridMapPclLoader.cpp
  src/helpers.cpp
  src/PclLoaderParameters.cpp
  src/PointcloudProcessor.cpp
)

# Find OpenMP (optional for parallelization)
find_package(OpenMP QUIET)
if (OpenMP_FOUND)
  add_compile_options("${OpenMP_CXX_FLAGS}")
  add_definitions(-DGRID_MAP_PCL_OPENMP_FOUND=${OpenMP_FOUND})
endif()

# Find Eigen3
find_package(Eigen3 REQUIRED)

# Find PCL
find_package(PCL REQUIRED
  COMPONENTS
    common
    features
    filters
    io
    kdtree
    segmentation
    surface
)

# Find yaml-cpp
find_package(yaml-cpp REQUIRED)

# Find grid_map_core (assuming it's installed or available)
find_package(grid_map_core REQUIRED)

###########
## Build ##
###########

# Library
add_library(${PROJECT_NAME}
  ${SRC_FILES}
)

target_include_directories(${PROJECT_NAME} PRIVATE
  include
)

target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
  ${EIGEN3_INCLUDE_DIR}
  ${OpenMP_CXX_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${YAML_CPP_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}
  grid_map_core
  ${OpenMP_CXX_LIBRARIES}
  ${PCL_LIBRARIES}
  yaml-cpp
)

# Set library properties for installation
set_target_properties(${PROJECT_NAME} PROPERTIES
  VERSION 1.0.0
  SOVERSION 1
  PUBLIC_HEADER "include/${PROJECT_NAME}/GridMapPclConverter.hpp;include/${PROJECT_NAME}/GridMapPclLoader.hpp;include/${PROJECT_NAME}/helpers.hpp;include/${PROJECT_NAME}/PclLoaderParameters.hpp;include/${PROJECT_NAME}/PointcloudProcessor.hpp"
)

#############
## Install ##
#############

include(GNUInstallDirs)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}Targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
  FILES_MATCHING PATTERN "*.hpp"
)

install(
  DIRECTORY config doc
  DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}
)

install(
  FILES README.md
  DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}
)

# Export targets for downstream projects
install(
  EXPORT ${PROJECT_NAME}Targets
  FILE ${PROJECT_NAME}Targets.cmake
  NAMESPACE grid_map::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

# Create and install package config file
include(CMakePackageConfigHelpers)

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION 1.0.0
  COMPATIBILITY SameMajorVersion
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

#############
## Testing ##
#############

option(BUILD_TESTING "Build tests" OFF)

if(BUILD_TESTING)
  enable_testing()
  find_package(GTest REQUIRED)

  add_executable(${PROJECT_NAME}_test
    test/test_grid_map_pcl.cpp
    test/GridMapPclLoaderTest.cpp
    test/HelpersTest.cpp
    test/PointcloudProcessorTest.cpp
    test/test_helpers.cpp
    test/PointcloudCreator.cpp
  )

  target_include_directories(${PROJECT_NAME}_test PRIVATE
    include
  )

  target_include_directories(${PROJECT_NAME}_test SYSTEM PUBLIC
    ${EIGEN3_INCLUDE_DIR}
    ${OpenMP_CXX_INCLUDE_DIRS}
  )

  target_link_libraries(${PROJECT_NAME}_test
    ${PROJECT_NAME}
    GTest::GTest
    GTest::Main
    pthread
  )

  include(GoogleTest)
  gtest_discover_tests(${PROJECT_NAME}_test)
endif()

# Optional: Build example executables (if needed)
option(BUILD_EXAMPLES "Build example programs" OFF)

if(BUILD_EXAMPLES)
  add_executable(grid_map_pcl_loader_example
    examples/grid_map_pcl_loader_example.cpp
  )
  
  target_include_directories(grid_map_pcl_loader_example PRIVATE
    include
  )
  
  target_link_libraries(grid_map_pcl_loader_example
    ${PROJECT_NAME}
  )
  
  install(TARGETS grid_map_pcl_loader_example
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
endif()
