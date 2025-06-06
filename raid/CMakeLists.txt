# raid/CMakeLists.txt

set(RAID_SOURCES
    raid_base.c
)

set(RAID_SOURCES_X86_64
    pq_gen_sse.asm
    pq_gen_avx.asm
    pq_gen_avx2.asm
    pq_gen_avx512.asm
    pq_check_sse.asm
    # GFNI specific files, conditionally added
    xor_gen_sse.asm
    xor_gen_avx.asm
    xor_gen_avx512.asm
    xor_check_sse.asm
    raid_multibinary.asm
)

set(RAID_SOURCES_X86_64_GFNI
    pq_gen_avx2_gfni.asm
    pq_gen_avx512_gfni.asm
)

set(RAID_SOURCES_AARCH64
    aarch64/raid_aarch64_dispatcher.c
    aarch64/raid_multibinary_arm.S
    aarch64/pq_gen_neon.S
    aarch64/pq_check_neon.S
    aarch64/xor_gen_neon.S
    aarch64/xor_check_neon.S
)

set(RAID_SOURCES_RISCV64
    riscv64/raid_multibinary_riscv64_dispatcher.c
    riscv64/raid_multibinary_riscv64.S
    # Add other riscv64 specific files from raid/riscv64/Makefile.am
)

set(RAID_SOURCES_RISCV64_RVV
    riscv64/raid_pq_gen_rvv.S
    riscv64/raid_xor_gen_rvv.S
)

# Add the library target for the raid module
add_library(raid OBJECT ${RAID_SOURCES})

target_include_directories(raid PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_include_directories(raid PRIVATE ${CMAKE_SOURCE_DIR}/include)

if(TARGET_ARCH STREQUAL "x86_64")
    target_sources(raid PRIVATE ${RAID_SOURCES_X86_64})
    # Assuming ENABLE_AVX512 option is available from parent to control GFNI/AVX512 specific code
    # This might need a specific GFNI check or option.
    # Using ENABLE_AVX512 as a proxy for now, as GFNI often comes with AVX512.
    if(ENABLE_AVX512) # This should ideally be a specific check for GFNI
        target_sources(raid PRIVATE ${RAID_SOURCES_X86_64_GFNI})
        target_compile_definitions(raid PRIVATE HAVE_GFNI) # Example define
    endif()
    target_sources(raid PRIVATE raid_base_aliases.c) # From lsrc_base_aliases in autotools
elif(TARGET_ARCH STREQUAL "aarch64")
    target_sources(raid PRIVATE ${RAID_SOURCES_AARCH64})
    target_compile_definitions(raid PRIVATE ISA_L_AARCH64)
elif(TARGET_ARCH STREQUAL "riscv64")
    target_sources(raid PRIVATE ${RAID_SOURCES_RISCV64})
    if(HAVE_RVV)
        target_sources(raid PRIVATE ${RAID_SOURCES_RISCV64_RVV})
        target_compile_definitions(raid PRIVATE HAVE_RVV)
    endif()
    target_compile_definitions(raid PRIVATE ISA_L_RISCV64)
else()
    # Fallback for undefined or generic architecture (CPU_UNDEFINED in autotools)
    target_sources(raid PRIVATE raid_base_aliases.c)
endif()

# Handle .S files for AArch64 and RISC-V (compile as C for preprocessing)
if(TARGET_ARCH STREQUAL "aarch64" OR TARGET_ARCH STREQUAL "riscv64")
    set(ARCH_S_FILES "")
    if(TARGET_ARCH STREQUAL "aarch64")
        list(APPEND ARCH_S_FILES ${RAID_SOURCES_AARCH64})
    else() # riscv64
        list(APPEND ARCH_S_FILES ${RAID_SOURCES_RISCV64})
        if(HAVE_RVV)
            list(APPEND ARCH_S_FILES ${RAID_SOURCES_RISCV64_RVV})
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

# Create raid_base_aliases.c if it doesn't exist to prevent build error
# In autotools, this seems to be conditionally compiled.
# For CMake, ensure it's available or handle its absence gracefully.
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raid_base_aliases.c")
    file(WRITE "${CMAKE_CURRENT_SOURCE_DIR}/raid_base_aliases.c" "// Placeholder for raid_base_aliases.c
#include <raid.h>
// Add minimal required symbols or functions if any, or leave empty if it's truly optional and handled by ifdefs
")
endif()
