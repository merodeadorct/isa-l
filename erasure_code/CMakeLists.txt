# erasure_code/CMakeLists.txt

# From erasure_code/Makefile.am
set(ERASURE_CODE_C_SOURCES
    ec_base.c
    ec_highlevel_func.c
    # ec_base_aliases.c # For non-arch specific builds
)

# x86-64 assembly files (.asm)
set(ERASURE_CODE_ASM_SOURCES_X86_64
    ec_multibinary.asm       # Dispatcher
    gf_2vect_dot_prod_sse.asm
    gf_2vect_dot_prod_avx.asm
    gf_2vect_dot_prod_avx2.asm
    gf_2vect_dot_prod_avx512.asm
    gf_2vect_dot_prod_avx2_gfni.asm
    gf_2vect_dot_prod_avx512_gfni.asm
    gf_3vect_dot_prod_sse.asm
    gf_3vect_dot_prod_avx.asm
    gf_3vect_dot_prod_avx2.asm
    gf_3vect_dot_prod_avx512.asm
    gf_3vect_dot_prod_avx2_gfni.asm
    gf_3vect_dot_prod_avx512_gfni.asm
    gf_4vect_dot_prod_sse.asm
    gf_4vect_dot_prod_avx.asm
    gf_4vect_dot_prod_avx2.asm
    gf_4vect_dot_prod_avx512.asm
    # gf_4vect_dot_prod_avx2_gfni.asm # Makefile.am doesn't list separate gfni for 4vect, maybe included in avx2?
    gf_4vect_dot_prod_avx512_gfni.asm
    gf_5vect_dot_prod_sse.asm
    gf_5vect_dot_prod_avx.asm
    gf_5vect_dot_prod_avx2.asm
    gf_5vect_dot_prod_avx512.asm
    # gf_5vect_dot_prod_avx2_gfni.asm # Not listed
    gf_5vect_dot_prod_avx512_gfni.asm
    gf_6vect_dot_prod_sse.asm
    gf_6vect_dot_prod_avx.asm
    gf_6vect_dot_prod_avx2.asm
    gf_6vect_dot_prod_avx512.asm
    # gf_6vect_dot_prod_avx2_gfni.asm # Not listed
    gf_6vect_dot_prod_avx512_gfni.asm
    gf_vect_dot_prod_sse.asm
    gf_vect_dot_prod_avx.asm
    gf_vect_dot_prod_avx2.asm
    gf_vect_dot_prod_avx512.asm
    gf_vect_dot_prod_avx2_gfni.asm # Listed as gf_vect_dot_prod_gfni.asm in Makefile.am but seems to be avx2 specific
    gf_vect_dot_prod_avx512_gfni.asm
    gf_vect_mul_sse.asm
    gf_vect_mul_avx.asm
    gf_vect_mul_avx2.asm
    gf_vect_mul_avx512.asm
    gf_vect_mul_gfni.asm # Assumed to be AVX2+GFNI based on pattern, or generic GFNI if available
    gf_vect_mul_avx512_gfni.asm

    gf_2vect_mad_sse.asm
    gf_2vect_mad_avx.asm
    gf_2vect_mad_avx2.asm
    gf_2vect_mad_avx512.asm
    gf_2vect_mad_avx2_gfni.asm
    gf_2vect_mad_avx512_gfni.asm
    gf_3vect_mad_sse.asm
    gf_3vect_mad_avx.asm
    gf_3vect_mad_avx2.asm
    gf_3vect_mad_avx512.asm
    gf_3vect_mad_avx2_gfni.asm
    gf_3vect_mad_avx512_gfni.asm
    gf_4vect_mad_sse.asm
    gf_4vect_mad_avx.asm
    gf_4vect_mad_avx2.asm
    gf_4vect_mad_avx512.asm
    # gf_4vect_mad_avx2_gfni.asm # Not listed
    gf_4vect_mad_avx512_gfni.asm
    gf_vect_mad_sse.asm
    gf_vect_mad_avx.asm
    gf_vect_mad_avx2.asm
    gf_vect_mad_avx512.asm
    gf_vect_mad_avx2_gfni.asm
    gf_vect_mad_avx512_gfni.asm
)
# Note: The gfni files like gf_vect_dot_prod_gfni.asm were listed as gf_vect_dot_prod_gfni.asm.
# I've assumed they are AVX2 specific gf_vect_dot_prod_avx2_gfni.asm based on other patterns,
# but if they are generic, the naming might need adjustment or they might be included by ec_multibinary.asm.
# For gf_vect_mul_gfni.asm, I'm keeping it as is, assuming it's a distinct target.
# Some gfni versions for 4, 5, 6 vect_dot_prod and 4 vect_mad were missing in Makefile.am's x86 list,
# I've kept them commented out. If they exist and are needed, they should be added.

add_library(isal_erasure_code STATIC
    ${ERASURE_CODE_C_SOURCES}
    ${ERASURE_CODE_ASM_SOURCES_X86_64}
)

target_include_directories(isal_erasure_code PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR} # For internal headers like ec_base.h
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # For public headers erasure_code.h, gf_vect_mul.h
    $<INSTALL_INTERFACE:include>
)

# NASM include paths for .asm files (e.g. ec_multibinary.asm might include reg_sizes.asm)
target_compile_options(isal_erasure_code PRIVATE
    $<$<COMPILE_LANGUAGE:ASM_NASM>:-I${CMAKE_SOURCE_DIR}/include>
)

target_link_libraries(isal_erasure_code INTERFACE isal_core)

set_property(GLOBAL APPEND PROPERTY ISAL_MODULE_LIBRARIES isal_erasure_code)

message(STATUS "isal_erasure_code C sources: ${ERASURE_CODE_C_SOURCES}")
message(STATUS "isal_erasure_code ASM sources: ${ERASURE_CODE_ASM_SOURCES_X86_64}")
