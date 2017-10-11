cmake_minimum_required(VERSION 3.5)
project(tfexample)

set(CMAKE_CXX_STANDARD 11)

# add modules
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/Modules")

# find packages
find_package(TensorFlow REQUIRED)
find_package(Eigen REQUIRED)
find_package(Protobuf REQUIRED)

set(PROJECT_INCLUDE_DIRS ${TensorFlow_INCLUDE_DIRS} ${Eigen_INCLUDE_DIRS} ${PROTOBUF_INCLUDE_DIRS})
set(PROJECT_LIBRARIES ${TensorFlow_LIBRARIES} ${PROTOBUF_LIBRARIES})
include_directories(${PROJECT_INCLUDE_DIRS})

set(SOURCE_FILES example_trainer.cc)
set(EXECUTABLE tfexample)
add_executable(${EXECUTABLE} ${SOURCE_FILES})
target_link_libraries(${EXECUTABLE} ${PROJECT_LIBRARIES})