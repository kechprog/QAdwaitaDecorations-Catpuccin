cmake_minimum_required(VERSION 3.5)
project(TestQtApp)

set(CMAKE_CXX_STANDARD 11)

find_package(Qt6 REQUIRED COMPONENTS Widgets)

add_executable(TestQtApp main.cpp)

target_link_libraries(TestQtApp Qt6::Widgets)
