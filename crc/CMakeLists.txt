# crc/CMakeLists.txt

set(CRC_SOURCES
    crc_base.c # Common CRC functions, like crc16_t10dif_generic, etc.
    # Other common C files from crc/Makefile.am (lsrc)
    # e.g., crc16_t10dif_copy_perf.c, crc_reflect.c etc. are test/perf, not core lib sources.
    # crc64_base.c is listed in crc/Makefile.am
    crc64_base.c
)

set(CRC_SOURCES_X86_64
    # CRC16
    crc16_t10dif_01.asm
    crc16_t10dif_02.asm
    crc16_t10dif_by4.asm
    crc16_t10dif_copy_by4.asm
    crc16_t10dif_copy_by4_02.asm
    crc16_t10dif_by16_10.asm
    # CRC32 GZIP
    crc32_gzip_refl_by8.asm
    crc32_gzip_refl_by8_02.asm
    crc32_gzip_refl_by16_10.asm
    # CRC32 IEEE
    crc32_ieee_01.asm
    crc32_ieee_02.asm
    crc32_ieee_by4.asm
    crc32_ieee_by16_10.asm
    # CRC32 ISCSI
    crc32_iscsi_00.asm
    crc32_iscsi_01.asm
    crc32_iscsi_by16_10.asm
    # CRC64
    crc64_ecma_norm_by8.asm
    crc64_ecma_norm_by16_10.asm
    crc64_ecma_refl_by8.asm
    crc64_ecma_refl_by16_10.asm
    crc64_iso_norm_by8.asm
    crc64_iso_norm_by16_10.asm
    crc64_iso_refl_by8.asm
    crc64_iso_refl_by16_10.asm
    crc64_jones_norm_by8.asm
    crc64_jones_norm_by16_10.asm
    crc64_jones_refl_by8.asm
    crc64_jones_refl_by16_10.asm
    crc64_rocksoft_norm_by8.asm
    crc64_rocksoft_norm_by16_10.asm
    crc64_rocksoft_refl_by8.asm
    crc64_rocksoft_refl_by16_10.asm
    # Multibinary files
    crc_multibinary.asm
    crc64_multibinary.asm
)

set(CRC_SOURCES_AARCH64
    aarch64/crc_aarch64_dispatcher.c
    aarch64/crc_multibinary_arm.S
    # CRC16
    aarch64/crc16_t10dif_pmull.S
    aarch64/crc16_t10dif_copy_pmull.S
    # CRC32
    aarch64/crc32_gzip_refl_pmull.S
    aarch64/crc32_gzip_refl_crc_ext.S
    aarch64/crc32_ieee_norm_pmull.S
    aarch64/crc32_iscsi_refl_pmull.S
    aarch64/crc32_iscsi_crc_ext.S
    # CRC32 fold/mix variants
    aarch64/crc32_gzip_refl_3crc_fold.S
    aarch64/crc32_iscsi_3crc_fold.S
    aarch64/crc32_common_crc_ext_cortex_a72.S # common mix file
    aarch64/crc32_mix_default.S
    aarch64/crc32c_mix_default.S
    aarch64/crc32_common_mix_neoverse_n1.S # common mix file
    aarch64/crc32_mix_neoverse_n1.S
    aarch64/crc32c_mix_neoverse_n1.S
    # CRC64
    aarch64/crc64_ecma_norm_pmull.S
    aarch64/crc64_ecma_refl_pmull.S
    aarch64/crc64_iso_norm_pmull.S
    aarch64/crc64_iso_refl_pmull.S
    aarch64/crc64_jones_norm_pmull.S
    aarch64/crc64_jones_refl_pmull.S
    aarch64/crc64_rocksoft_norm_pmull.S
    aarch64/crc64_rocksoft_refl_pmull.S
)

set(CRC_SOURCES_PPC64LE
    ppc64le/crc_ppc64le_dispatcher.c
    ppc64le/crc_multibinary_ppc.S
    # CRC32
    ppc64le/crc32_iscsi_vpmsum.S
    ppc64le/crc32_ieee_vpmsum.S
    # CRC16
    ppc64le/crc16_t10dif_vpmsum.S
    # Add other ppc64le specific files from crc/ppc64le/Makefile.am if any
)

set(CRC_SOURCES_RISCV64
    riscv64/crc_riscv64_dispatcher.c
    riscv64/crc_multibinary_riscv64.S
)
set(CRC_SOURCES_RISCV64_RVV
    riscv64/crc32_ieee_norm_rvv.S
    riscv64/crc32_iscsi_refl_rvv.S
    riscv64/crc16_t10dif_rvv.S
)


# Add the library target for the crc module
add_library(crc OBJECT ${CRC_SOURCES})

target_include_directories(crc PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_include_directories(crc PRIVATE ${CMAKE_SOURCE_DIR}/include)

if(TARGET_ARCH STREQUAL "x86_64")
    target_sources(crc PRIVATE ${CRC_SOURCES_X86_64})
    target_sources(crc PRIVATE crc_base_aliases.c) # From lsrc_base_aliases in autotools
elif(TARGET_ARCH STREQUAL "aarch64")
    target_sources(crc PRIVATE ${CRC_SOURCES_AARCH64})
    target_compile_definitions(crc PRIVATE ISA_L_AARCH64)
elif(TARGET_ARCH STREQUAL "ppc64le")
    target_sources(crc PRIVATE ${CRC_SOURCES_PPC64LE})
    target_compile_definitions(crc PRIVATE ISA_L_PPC64LE)
elif(TARGET_ARCH STREQUAL "riscv64")
    target_sources(crc PRIVATE ${CRC_SOURCES_RISCV64})
    if(HAVE_RVV)
        target_sources(crc PRIVATE ${CRC_SOURCES_RISCV64_RVV})
        target_compile_definitions(crc PRIVATE HAVE_RVV)
    endif()
    target_compile_definitions(crc PRIVATE ISA_L_RISCV64)
else()
    # Fallback for undefined or generic architecture
    target_sources(crc PRIVATE crc_base_aliases.c)
endif()

# Handle .S files for AArch64, PPC64LE and RISC-V (compile as C for preprocessing)
if(TARGET_ARCH STREQUAL "aarch64" OR TARGET_ARCH STREQUAL "ppc64le" OR TARGET_ARCH STREQUAL "riscv64")
    set(ARCH_S_FILES "")
    if(TARGET_ARCH STREQUAL "aarch64")
        list(APPEND ARCH_S_FILES ${CRC_SOURCES_AARCH64})
    elseif(TARGET_ARCH STREQUAL "ppc64le")
        list(APPEND ARCH_S_FILES ${CRC_SOURCES_PPC64LE})
    else() # riscv64
        list(APPEND ARCH_S_FILES ${CRC_SOURCES_RISCV64})
        if(HAVE_RVV)
            list(APPEND ARCH_S_FILES ${CRC_SOURCES_RISCV64_RVV})
        endif()
    endif()

    foreach(S_FILE ${ARCH_S_FILES})
        if (S_FILE MATCHES "\\.S$")
            set_source_files_properties(${S_FILE} PROPERTIES LANGUAGE C)
            if(TARGET_ARCH STREQUAL "aarch64")
                 set_source_files_properties(${S_FILE} PROPERTIES COMPILE_DEFINITIONS "__ASSEMBLY__")
            endif()
        endif()
    endforeach()
endif()

# Create crc_base_aliases.c if it doesn't exist
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/crc_base_aliases.c")
    file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/crc_base_aliases.c" "// Placeholder for crc_base_aliases.c
#include <crc.h>
#include <crc_ref.h>
// Add minimal required symbols or functions if any
")
endif()
# Create crc_ref.h if it doesn't exist, as crc_base_aliases.c might include it.
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/crc_ref.h")
    file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/crc_ref.h" "// Placeholder for crc_ref.h
#ifndef CRC_REF_H
#define CRC_REF_H
#include <stdint.h>
// Define any referenced symbols if necessary
#endif
")
endif()
