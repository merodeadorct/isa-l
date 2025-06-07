# mem/CMakeLists.txt

# From mem/Makefile.am
set(MEM_C_SOURCES
    mem_zero_detect_base.c
    # mem_zero_detect_base_aliases.c # For non-arch specific builds
)

# x86-64 assembly files (.asm)
set(MEM_ASM_SOURCES_X86_64
    mem_multibinary.asm       # Dispatcher
    mem_zero_detect_sse.asm
    mem_zero_detect_avx.asm   # Not explicitly in Makefile.am's lsrc_x86_64, but often SSE implies AVX, AVX2 etc. are available.
                              # mem_zero_detect_avx.asm is present in the directory.
    mem_zero_detect_avx2.asm
    mem_zero_detect_avx512.asm
)
# Note: mem/Makefile.am only explicitly lists:
#   mem_zero_detect_sse.asm
#   mem_zero_detect_avx2.asm
#   mem_zero_detect_avx512.asm
#   mem_multibinary.asm
# However, mem_zero_detect_avx.asm exists. It's possible it's called by mem_multibinary.asm or
# was omitted from the list. Including it for completeness as it's an x86 feature level.

add_library(isal_mem STATIC
    ${MEM_C_SOURCES}
    ${MEM_ASM_SOURCES_X86_64}
)

target_include_directories(isal_mem PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR} # For internal headers if any
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # For public header mem_routines.h
    $<INSTALL_INTERFACE:include>
)

# NASM include paths for .asm files (e.g. mem_multibinary.asm might include reg_sizes.asm)
target_compile_options(isal_mem PRIVATE
    $<$<COMPILE_LANGUAGE:ASM_NASM>:-I${CMAKE_SOURCE_DIR}/include>
)

target_link_libraries(isal_mem INTERFACE isal_core)

set_property(GLOBAL APPEND PROPERTY ISAL_MODULE_LIBRARIES isal_mem)

message(STATUS "isal_mem C sources: ${MEM_C_SOURCES}")
message(STATUS "isal_mem ASM sources: ${MEM_ASM_SOURCES_X86_64}")
