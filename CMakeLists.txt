cmake_minimum_required(VERSION 3.21)

project(sharing_session)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_subdirectory(async)
add_subdirectory(concept)

add_executable(main main.cpp)

target_link_libraries(main PRIVATE async concept)
