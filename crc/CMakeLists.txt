# crc/CMakeLists.txt

# List C source files for the crc module
# From crc/Makefile.am:
# lsrc  += crc/crc_base.c crc/crc64_base.c
# lsrc_base_aliases += crc/crc_base_aliases.c (used if CPU_UNDEFINED)
# For x86-64, we don't use crc_base_aliases.c directly in the main lib,
# but it might be used by some specific configurations or tests.
# For now, focusing on the core library sources for x86.
set(CRC_C_SOURCES
    crc_base.c
    crc64_base.c
)

# List x86-64 assembly files (.asm) for the crc module
# From crc/Makefile.am (lsrc_x86_64 section):
set(CRC_ASM_SOURCES_X86_64
    crc16_t10dif_01.asm
    crc16_t10dif_by4.asm
    crc16_t10dif_02.asm
    crc16_t10dif_by16_10.asm
    crc16_t10dif_copy_by4.asm
    crc16_t10dif_copy_by4_02.asm
    crc32_ieee_01.asm
    crc32_ieee_02.asm
    crc32_ieee_by4.asm
    crc32_ieee_by16_10.asm
    crc32_iscsi_01.asm
    crc32_iscsi_00.asm
    crc32_iscsi_by16_10.asm
    crc_multibinary.asm       # This likely contains dispatchers / CPU feature detection
    crc64_multibinary.asm     # Similar to above for crc64
    crc64_ecma_refl_by8.asm
    crc64_ecma_refl_by16_10.asm
    crc64_ecma_norm_by8.asm
    crc64_ecma_norm_by16_10.asm
    crc64_iso_refl_by8.asm
    crc64_iso_refl_by16_10.asm
    crc64_iso_norm_by8.asm
    crc64_iso_norm_by16_10.asm
    crc64_jones_refl_by8.asm
    crc64_jones_refl_by16_10.asm
    crc64_jones_norm_by8.asm
    crc64_jones_norm_by16_10.asm
    crc64_rocksoft_refl_by8.asm
    crc64_rocksoft_refl_by16_10.asm
    crc64_rocksoft_norm_by8.asm
    crc64_rocksoft_norm_by16_10.asm
    crc32_gzip_refl_by8.asm
    crc32_gzip_refl_by8_02.asm
    crc32_gzip_refl_by16_10.asm
)

# AArch64 sources are in crc/aarch64, not including them for this x86-only build for now.
# crc/aarch64/crc_aarch64_dispatcher.c
# crc/aarch64/crc_common_pmull.h (and other .S files)

# Create a static library for the crc module
add_library(isal_crc STATIC
    ${CRC_C_SOURCES}
    ${CRC_ASM_SOURCES_X86_64}
)

# Set include directories for this module
# src_include += -I $(srcdir)/crc (from crc/Makefile.am)
# Global include is already in root CMakeLists.txt
target_include_directories(isal_crc PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR} # For internal headers like crc_ref.h, crc64_ref.h if needed by other modules
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # For public headers like crc.h, crc64.h
    $<INSTALL_INTERFACE:include> # For installed headers
)

# Link this module library to the main isal_core interface target
# This makes its public include directories available to other modules if needed
# and prepares for linking into the final shared library
target_link_libraries(isal_crc INTERFACE isal_core)

# Add this library to the list of libraries that make up the core 'isal' library.
# We'll use a global property or variable to collect these.
# This is one way to manage the list of component libraries.
# The root CMakeLists.txt will then use this list to link the final shared library.
# Add this library to a global property to be collected by the root CMakeLists.txt
set_property(GLOBAL APPEND PROPERTY ISAL_MODULE_LIBRARIES isal_crc)

message(STATUS "isal_crc sources: ${CRC_C_SOURCES} ${CRC_ASM_SOURCES_X86_64}")
