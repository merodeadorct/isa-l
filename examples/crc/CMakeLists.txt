# examples/crc/CMakeLists.txt
if(NOT BUILD_EXAMPLES)
    return()
endif()

# Assuming add_isal_example is defined in the parent scope (examples/CMakeLists.txt)
# If not, it needs to be redefined or included. For simplicity, assume it's available.

add_isal_example(crc_combine_example crc_combine_example.c)
# crc64_example from main crc/Makefile.am's example list
add_isal_example(crc64_example ../../crc/crc64_example.c ../../crc/crc64_ref.h)

# Create placeholder for crc64_ref.h if it doesn't exist in the main crc directory
# This is also handled in tests/CMakeLists.txt, but good to ensure it here too if tests are not built.
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/crc/crc64_ref.h")
    file(WRITE "${CMAKE_SOURCE_DIR}/crc/crc64_ref.h" "// Placeholder for crc64_ref.h
#ifndef CRC64_REF_H
#define CRC64_REF_H
#include <stdint.h>
#endif
")
endif()
