# raid/CMakeLists.txt

# From raid/Makefile.am
set(RAID_C_SOURCES
    raid_base.c
    # raid_base_aliases.c # For non-arch specific builds
)

# x86-64 assembly files (.asm)
set(RAID_ASM_SOURCES_X86_64
    raid_multibinary.asm  # Dispatcher
    pq_check_sse.asm
    pq_gen_sse.asm
    pq_gen_avx.asm
    pq_gen_avx2.asm
    pq_gen_avx512.asm
    pq_gen_avx2_gfni.asm
    pq_gen_avx512_gfni.asm
    xor_check_sse.asm
    xor_gen_sse.asm
    xor_gen_avx.asm
    # xor_gen_avx2.asm # Makefile.am lists xor_gen_avx, xor_gen_avx512. AVX2 might be covered by AVX or AVX512 specific versions or not present.
    xor_gen_avx512.asm
)
# Note: xor_gen_avx2.asm was not explicitly listed under lsrc_x86_64 in the top Makefile.am for raid.
# It's possible it's not built or is implicitly included/selected.
# For now, keeping it commented out based on the explicit list.

add_library(isal_raid STATIC
    ${RAID_C_SOURCES}
    ${RAID_ASM_SOURCES_X86_64}
)

target_include_directories(isal_raid PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR} # For internal headers if any
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # For public header raid.h
    $<INSTALL_INTERFACE:include>
)

# NASM include paths for .asm files (e.g. raid_multibinary.asm might include reg_sizes.asm)
target_compile_options(isal_raid PRIVATE
    $<$<COMPILE_LANGUAGE:ASM_NASM>:-I${CMAKE_SOURCE_DIR}/include>
)

target_link_libraries(isal_raid INTERFACE isal_core)

set_property(GLOBAL APPEND PROPERTY ISAL_MODULE_LIBRARIES isal_raid)

message(STATUS "isal_raid C sources: ${RAID_C_SOURCES}")
message(STATUS "isal_raid ASM sources: ${RAID_ASM_SOURCES_X86_64}")
