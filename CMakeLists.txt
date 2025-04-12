cmake_minimum_required(VERSION 3.14)
project(ninja_lib)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# List of source files needed for ninja as a library
set(NINJA_LIB_SOURCES
	src/build_log.cc
	src/build.cc
	src/clean.cc
	src/clparser.cc
	src/dyndep.cc
	src/dyndep_parser.cc
	src/debug_flags.cc
	src/deps_log.cc
	src/disk_interface.cc
	src/edit_distance.cc
	src/elide_middle.cc
	src/eval_env.cc
	src/graph.cc
	src/graphviz.cc
	src/json.cc
	src/line_printer.cc
	src/manifest_parser.cc
	src/metrics.cc
	src/missing_deps.cc
	src/parser.cc
	src/real_command_runner.cc
	src/state.cc
	src/status_printer.cc
	src/string_piece_util.cc
	src/util.cc
	src/version.cc
	src/lexer.cc          
    src/depfile_parser.cc 
)

# Create a library target
if(BUILD_NINJA_SHARED)
    add_library(ninja_lib SHARED ${NINJA_LIB_SOURCES})
    # Define export macro for shared library
    target_compile_definitions(ninja_lib PRIVATE NINJA_EXPORT)
    # Set version for shared library
    set_target_properties(ninja_lib PROPERTIES
        VERSION 1.0.0
        SOVERSION 1
    )
else()
    add_library(ninja_lib STATIC ${NINJA_LIB_SOURCES})
endif()

# Add include directories
target_include_directories(ninja_lib
    PUBLIC 
        ${CMAKE_CURRENT_SOURCE_DIR}/src
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# Handle platform-specific dependencies
if(WIN32)
    target_sources(ninja_lib PRIVATE
		src/subprocess-win32.cc
		src/includes_normalize-win32.cc
		src/msvc_helper-win32.cc
		src/msvc_helper_main-win32.cc
		src/getopt.c
		src/minidump-win32.cc
    )
    target_link_libraries(ninja_lib PRIVATE shlwapi)
# else()
# 	target_sources(libninja PRIVATE src/subprocess-posix.cc)
# 	if(CMAKE_SYSTEM_NAME STREQUAL "OS400" OR CMAKE_SYSTEM_NAME STREQUAL "AIX")
# 		target_sources(libninja PRIVATE src/getopt.c)
# 		# Build getopt.c, which can be compiled as either C or C++, as C++
# 		# so that build environments which lack a C compiler, but have a C++
# 		# compiler may build ninja.
# 		set_source_files_properties(src/getopt.c PROPERTIES LANGUAGE CXX)
# 	endif()

# 	# Needed for perfstat_cpu_total
# 	if(CMAKE_SYSTEM_NAME STREQUAL "AIX")
# 		target_link_libraries(libninja PUBLIC "-lperfstat")
# 	endif()

endif()

# Export include directories for projects that use this library
target_include_directories(ninja_lib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include/ninja>
)

# Install rules for the library and headers
install(TARGETS ninja_lib
    EXPORT ninja_lib_targets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include/ninja
)

# Install public headers
install(DIRECTORY src/
    DESTINATION include/ninja
    FILES_MATCHING PATTERN "*.h"
)