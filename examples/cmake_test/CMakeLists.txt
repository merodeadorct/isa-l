# examples/cmake_test/CMakeLists.txt
cmake_minimum_required(VERSION 3.12)
project(isal_ec_simple_example C)

if(NOT TARGET isal)
    message(FATAL_ERROR "The 'isal' library target is not defined. This test must be built after the main library.")
endif()

add_executable(ec_simple_example ec_simple_example.c)

# Link against the isal library built by the parent project
# The isal library target should provide its include directories via target_include_directories
target_link_libraries(ec_simple_example PRIVATE isal)

# Add this executable as a test
enable_testing()
add_test(NAME ECExampleTest COMMAND ec_simple_example)

message(STATUS "Added EC simple example executable and test.")
