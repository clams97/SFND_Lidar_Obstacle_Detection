cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

if(POLICY CMP0144)
  cmake_policy(SET CMP0144 NEW)  # find_package uses upper-case <PACKAGE>_ROOT variables
endif()

project(lidar_obstacle_detection LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)  # build/compile_commands.json for IntelliSense

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

add_compile_options(-Wall -Wextra)

find_package(PCL 1.11 REQUIRED COMPONENTS common io filters kdtree search segmentation sample_consensus visualization)

# Shared rendering code, compiled once and reused by every program
add_library(render STATIC src/render/render.cpp)
target_include_directories(render PUBLIC ${PROJECT_SOURCE_DIR}/src ${PCL_INCLUDE_DIRS})
target_compile_definitions(render PUBLIC ${PCL_DEFINITIONS}
  DATA_DIR="${PROJECT_SOURCE_DIR}/src/sensors/data/pcd")
target_link_libraries(render PUBLIC ${PCL_LIBRARIES})

# Main project
add_executable(environment src/environment.cpp)
target_link_libraries(environment PRIVATE render)

# Quizzes
option(BUILD_QUIZZES "Build the RANSAC and clustering quiz programs" ON)
if(BUILD_QUIZZES)
  add_subdirectory(src/quiz/ransac)
  add_subdirectory(src/quiz/cluster)
endif()
