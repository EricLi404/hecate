cmake_minimum_required(VERSION 3.13)
project(hecate)

set(CMAKE_CXX_STANDARD 14)

include_directories(include)
include_directories(include/hecate)


find_package(OpenCV)
include_directories(/usr/local/include)
include_directories(${OpenCV_INCLUDE_DIRS})

FILE(GLOB_RECURSE laji include/*.hpp)
FILE(GLOB_RECURSE CPP_FILE src/*.cpp)

add_executable(hecate ${CPP_FILE} ${laji})

target_link_libraries(hecate ${OpenCV_LIBS})