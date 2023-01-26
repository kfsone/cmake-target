cmake_minimum_required (VERSION 3.16)
project (cmake-py3 LANGUAGES C CXX)

find_package (Python3 REQUIRED Development)

add_executable (main main.cpp)
target_link_libraries (main Python3::Python)
