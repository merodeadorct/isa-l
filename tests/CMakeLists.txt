# tests/CMakeLists.txt
if(NOT BUILD_TESTS)
    return()
endif()

enable_testing() # Enables CTest

# Common test utilities or settings can be defined here.
# For example, a function to simplify adding test executables.

# Add subdirectories for specific test categories
add_subdirectory(fuzz)

# Individual test programs from various modules
# These are based on check_PROGRAMS in Makefile.am and other test lists.

# Helper function to create and link a test
function(add_isal_test test_name)
    add_executable(${test_name} ${ARGN})
    target_link_libraries(${test_name} PRIVATE isal_static) # Changed from isal
    add_test(NAME ${test_name} COMMAND ${test_name})
    # Add labels for tests if desired, e.g., based on module
    # set_tests_properties(${test_name} PROPERTIES LABELS "module_X_test")
endfunction()

# CRC module tests
add_isal_test(crc_simple_test ../crc/crc_simple_test.c ../crc/crc_ref.h) # Assumes crc_ref.h is needed
add_isal_test(crc_byte_test ../crc/crc_byte_test.c ../crc/crc_ref.h)
add_isal_test(crc16_t10dif_test ../crc/crc16_t10dif_test.c ../crc/crc_ref.h)
add_isal_test(crc32_ieee_test ../crc/crc32_ieee_test.c ../crc/crc_ref.h)
add_isal_test(crc32_iscsi_test ../crc/crc32_iscsi_test.c ../crc/crc_ref.h)
add_isal_test(crc32_gzip_refl_test ../crc/crc32_gzip_refl_test.c ../crc/crc_ref.h)
add_isal_test(crc64_ecma_test ../crc/crc64_funcs_test.c ../crc/crc64_ref.h) # crc64_funcs_test covers multiple CRC64 types
    set_source_files_properties(../crc/crc64_funcs_test.c PROPERTIES COMPILE_DEFINITIONS "TEST_CRC64_ECMA")
add_isal_test(crc64_iso_test ../crc/crc64_funcs_test.c ../crc/crc64_ref.h)
    set_source_files_properties(../crc/crc64_funcs_test.c PROPERTIES COMPILE_DEFINITIONS "TEST_CRC64_ISO")
add_isal_test(crc64_jones_test ../crc/crc64_funcs_test.c ../crc/crc64_ref.h)
    set_source_files_properties(../crc/crc64_funcs_test.c PROPERTIES COMPILE_DEFINITIONS "TEST_CRC64_JONES")
add_isal_test(crc64_rocksoft_test ../crc/crc64_funcs_test.c ../crc/crc64_ref.h)
    set_source_files_properties(../crc/crc64_funcs_test.c PROPERTIES COMPILE_DEFINITIONS "TEST_CRC64_ROCKSOFT")
add_isal_test(crc_preserve_test ../crc/crc_preserve_test.c ../crc/crc_ref.h)
add_isal_test(crc_zeros_test ../crc/crc_zeros_test.c ../crc/crc_ref.h)
add_isal_test(crc_combine_test ../crc/crc_combine_test.c ../crc/crc_ref.h)
add_isal_test(crc_multibinary_test ../crc/crc_multibinary_test.c ../crc/crc_ref.h) # Generic test for multibinary crc
# crc64_multibinary_test uses the same source as crc_multibinary_test but with a different define
add_executable(crc64_multibinary_test ../crc/crc_multibinary_test.c ../crc/crc_ref.h)
target_compile_definitions(crc64_multibinary_test PRIVATE CRC64_TEST)
    target_link_libraries(crc64_multibinary_test PRIVATE isal_static) # Explicitly update here too
add_test(NAME crc64_multibinary_test COMMAND crc64_multibinary_test)


# Erasure Code module tests
add_isal_test(gf_vect_mul_test ../erasure_code/gf_vect_mul_test.c)
add_isal_test(erasure_code_test ../erasure_code/erasure_code_test.c)
add_isal_test(gf_inverse_test ../erasure_code/gf_inverse_test.c)
add_isal_test(erasure_code_update_test ../erasure_code/erasure_code_update_test.c)
# Unit tests (from unit_tests list in Makefile.am)
add_isal_test(gf_vect_mul_base_test ../erasure_code/gf_vect_mul_base_test.c)
add_isal_test(gf_vect_dot_prod_base_test ../erasure_code/gf_vect_dot_prod_base_test.c)
add_isal_test(gf_vect_dot_prod_test ../erasure_code/gf_vect_dot_prod_test.c)
add_isal_test(gf_vect_mad_test ../erasure_code/gf_vect_mad_test.c)
add_isal_test(erasure_code_base_test ../erasure_code/erasure_code_base_test.c)

# Igzip module tests
add_isal_test(igzip_stateless_test ../igzip/igzip_stateless_test.c)
add_isal_test(igzip_inflate_test ../igzip/igzip_inflate_test.c ../igzip/igzip_inflate_ref.h)
add_isal_test(huffman_test ../igzip/huffman_test.c)
add_isal_test(igzip_check_level_buf_test ../igzip/igzip_check_level_buf_test.c)
add_isal_test(checksum_test ../igzip/checksum32_funcs_test.c ../igzip/checksum_test_ref.h)
add_isal_test(igzip_multibinary_test ../igzip/igzip_multibinary_test.c)
add_isal_test(igzip_empty_input_test ../igzip/igzip_empty_input_test.c)


# Mem module tests
add_isal_test(mem_zero_detect_test ../mem/mem_zero_detect_test.c)

# Raid module tests
add_isal_test(xor_gen_test ../raid/xor_gen_test.c)
add_isal_test(xor_check_test ../raid/xor_check_test.c)
add_isal_test(pq_gen_test ../raid/pq_gen_test.c)
add_isal_test(pq_check_test ../raid/pq_check_test.c)

# Performance tests (can be added similarly if desired, or kept separate)
# Example: add_isal_test(crc16_t10dif_perf ../crc/crc16_t10dif_perf.c)
# For now, focusing on functional tests (check_tests, unit_tests)

# Create placeholder for crc_ref.h, crc64_ref.h, igzip_inflate_ref.h, checksum_test_ref.h if they don't exist in module directories
# These are included by test files.
set(MISSING_TEST_HEADERS
    ../crc/crc_ref.h
    ../crc/crc64_ref.h
    ../igzip/igzip_inflate_ref.h
    ../igzip/checksum_test_ref.h
)
foreach(header_file ${MISSING_TEST_HEADERS})
    if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${header_file}")
        file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/${header_file}" "// Placeholder for ${header_file}
#ifndef _PLACEHOLDER_H_
#define _PLACEHOLDER_H_
#include <stdint.h>
#endif
")
    endif()
endforeach()
