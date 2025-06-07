# igzip/CMakeLists.txt

# From igzip/Makefile.am
# General C sources
set(IGZIP_C_SOURCES
    adler32_base.c
    encode_df.c
    flatten_ll.c
    huff_codes.c
    hufftables_c.c
    igzip_base.c
    igzip_icf_base.c
    igzip_inflate.c
    igzip_stat_hist_base.c
    igzip_update_history_base.c
    # igzip.c # This seems to be for a command-line utility, not the core lib
    # igzip_base_aliases.c # For non-arch specific builds
    # igzip_icf_body.c # This has specific versions like igzip_icf_body_h1_gr_bt.asm
    # static_inflate_base.c # Seems to be related to igzip_inflate_multibinary.asm
)

# x86-64 specific C sources (if any, usually .asm or handled by multibinary)
# Most x86 specific logic is in .asm files or within C files guarded by ifdefs / function pointers

# x86-64 assembly files (.asm)
set(IGZIP_ASM_SOURCES_X86_64
    adler32_sse.asm
    adler32_avx2_4.asm # Higher level/optimized version
    bitbuf2.asm
    # data_struct2.asm # Seems to be an include or data definition for asm
    encode_df_04.asm
    encode_df_06.asm
    huffman.asm
    igzip_body.asm
    igzip_decode_block_stateless.asm
    igzip_decode_block_stateless_01.asm
    igzip_decode_block_stateless_04.asm
    igzip_deflate_hash.asm
    igzip_finish.asm
    igzip_gen_icf_map_lh1_04.asm
    igzip_gen_icf_map_lh1_06.asm
    igzip_icf_body_h1_gr_bt.asm # Example of specific ICF body
    # igzip_inflate_multibinary.asm # Handles dispatch for inflate
    # igzip_multibinary.asm # Handles dispatch for deflate
    igzip_process_match_table_avx2.asm
    igzip_process_match_table_avx512.asm
    igzip_stat_hist_avx2.asm
    igzip_stat_hist_avx512.asm
    igzip_update_history.asm
    # isal_deflate_body_avx2.asm
    # isal_deflate_body_avx512.asm
    # isal_deflate_finish_avx2.asm
    # isal_deflate_finish_avx512.asm
    # isal_deflate_icf_body_avx2.asm
    # isal_deflate_icf_body_avx512.asm
    # isal_deflate_icf_finish_avx2.asm
    # isal_deflate_icf_finish_avx512.asm
    # isal_fill_window_avx2.asm
    # isal_fill_window_sse.asm
    # isal_skip_avx2.asm
    # isal_skip_avx512.asm
    # isal_update_histogram_avx2.asm
    # isal_update_histogram_avx512.asm
    # slide_hash_avx2.asm
    # slide_hash_avx512.asm
    # slide_hash_sse.asm
    # write_huff_avx.asm
    # write_huff_avx2.asm
    # write_huff_avx2_fill.asm
    # write_huff_uncmp_avx2.asm
    # write_huff_uncmp_avx2_fill.asm

    # Multibinary files - these are key for dispatching
    igzip_inflate_multibinary.asm
    igzip_multibinary.asm
    isal_deflate_body_avx2.asm # These are functions called by dispatcher
    isal_deflate_body_avx512.asm
    isal_deflate_finish_avx2.asm
    isal_deflate_finish_avx512.asm
    isal_deflate_icf_body_avx2.asm
    isal_deflate_icf_body_avx512.asm
    isal_deflate_icf_finish_avx2.asm
    isal_deflate_icf_finish_avx512.asm
    isal_fill_window_avx2.asm
    isal_fill_window_sse.asm
    isal_skip_avx2.asm
    isal_skip_avx512.asm
    isal_update_histogram_avx2.asm
    isal_update_histogram_avx512.asm
    slide_hash_avx2.asm
    slide_hash_avx512.asm
    slide_hash_sse.asm
    write_huff_avx.asm
    write_huff_avx2.asm
    write_huff_avx2_fill.asm
    write_huff_uncmp_avx2.asm
    write_huff_uncmp_avx2_fill.asm
)
# Note: Some .asm files like data_struct2.asm might be included by other .asm files.
# NASM should handle these includes if the paths are correct.
# The `igzip/aarch64` sources are ignored for this x86 build.

add_library(isal_igzip STATIC
    ${IGZIP_C_SOURCES}
    ${IGZIP_ASM_SOURCES_X86_64}
)

target_include_directories(isal_igzip PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR} # For internal headers
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # For public headers like igzip_lib.h
    $<INSTALL_INTERFACE:include>
)

# NASM requires include paths for its %include directives.
# data_struct2.asm is in the current source dir.
# Other includes like reg_sizes.asm are in the top-level include dir.
target_compile_options(isal_igzip PRIVATE
    $<$<COMPILE_LANGUAGE:ASM_NASM>:-I${CMAKE_CURRENT_SOURCE_DIR} -I${CMAKE_SOURCE_DIR}/include>
)


target_link_libraries(isal_igzip INTERFACE isal_core)

set_property(GLOBAL APPEND PROPERTY ISAL_MODULE_LIBRARIES isal_igzip)

message(STATUS "isal_igzip C sources: ${IGZIP_C_SOURCES}")
message(STATUS "isal_igzip ASM sources: ${IGZIP_ASM_SOURCES_X86_64}")
