# programs/CMakeLists.txt

# Add igzip_cli program
add_executable(igzip_cli igzip_cli.c)
target_link_libraries(igzip_cli PRIVATE isal_static) # Changed from isal

install(TARGETS igzip_cli
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Add man page for igzip_cli
# The .1.h2m seems to be a source for a man page conversion tool.
# For CMake, we typically install pre-generated man pages or use a custom command.
# Assuming igzip.1 is the pre-generated man page.
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/igzip.1")
    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/igzip.1"
        DESTINATION ${CMAKE_INSTALL_MANDIR}/man1
    )
endif()

# Add other programs if any.
# The Makefile.am for programs only lists igzip_cli under bin_PROGRAMS.
# Tools like test_autorun.sh, etc., are scripts and not compiled programs.
