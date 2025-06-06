# tests/fuzz/CMakeLists.txt
if(NOT BUILD_TESTS) # Or a specific option like BUILD_FUZZ_TESTS
    return()
endif()

# Fuzz tests often require a fuzzing engine (like libFuzzer) which needs to be linked.
# This might require finding the fuzzer library or setting specific compiler flags (e.g., -fsanitize=fuzzer).
# For simplicity, we'll assume these are standard executables for now,
# or that the user will provide necessary flags via CMAKE_C_FLAGS.

# Helper function from parent scope might not be visible, redefine or include.
# Assuming add_isal_test is available or link manually.
function(add_isal_fuzz_test test_name)
    add_executable(${test_name} ${ARGN})
    target_link_libraries(${test_name} PRIVATE isal)
    # Fuzz tests are typically not added to 'ctest' in the same way as unit tests.
    # They are run with a fuzzing engine.
    # add_test(NAME ${test_name}_fuzz COMMAND ${test_name} <fuzzer_args>)
endfunction()

add_isal_fuzz_test(igzip_fuzz_inflate igzip_fuzz_inflate.c)
add_isal_fuzz_test(igzip_simple_inflate_fuzz_test igzip_simple_inflate_fuzz_test.c)
add_isal_fuzz_test(igzip_simple_round_trip_fuzz_test igzip_simple_round_trip_fuzz_test.c)
# igzip_checked_inflate_fuzz_test.c might be another one based on Makefile.am
add_isal_fuzz_test(igzip_checked_inflate_fuzz_test igzip_checked_inflate_fuzz_test.c)


# igzip_dump_inflate_corpus.c is a utility, not a test itself. Build if needed.
add_executable(igzip_dump_inflate_corpus igzip_dump_inflate_corpus.c)
target_link_libraries(igzip_dump_inflate_corpus PRIVATE isal)
