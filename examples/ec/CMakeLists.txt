# examples/ec/CMakeLists.txt
if(NOT BUILD_EXAMPLES)
    return()
endif()

# Assuming add_isal_example is defined in the parent scope

add_isal_example(ec_simple_example ec_simple_example.c)
add_isal_example(ec_piggyback_example ec_piggyback_example.c)
