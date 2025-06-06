# examples/CMakeLists.txt
if(NOT BUILD_EXAMPLES)
    return()
endif()

# Helper function to create and link an example program
function(add_isal_example example_name)
    add_executable(${example_name} ${ARGN})
    target_link_libraries(${example_name} PRIVATE isal)
    # Examples are not typically added to CTest unless they are self-checking
endfunction()

add_subdirectory(crc)
add_subdirectory(ec)

# Add other top-level examples if any.
# From igzip/Makefile.am, igzip_example.c and igzip_inflate_example.c are listed.
add_isal_example(igzip_example ../igzip/igzip_example.c)
add_isal_example(igzip_inflate_example ../igzip/igzip_inflate_example.c)

# From crc/Makefile.am, crc64_example.c is listed.
# This is already handled by examples/crc/CMakeLists.txt if we follow the pattern.
# Let's assume it will be there.

# From raid/Makefile.am, xor_example.c is listed.
add_isal_example(xor_example ../raid/xor_example.c)
