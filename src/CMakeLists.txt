
set(ALL_SRCS
	app.rc
	composition.h
	composition.cpp	
	d3d11.h
	d3d11.cpp
	image_layer.cpp
	web_layer.cpp
	main.cpp
	platform.h
	resource.h
	util.cpp
	util.h
)
	
# Logical target used to link the libcef library.
ADD_LOGICAL_TARGET("libcef_lib" "${CEF_LIB_DEBUG}" "${CEF_LIB_RELEASE}")

# Determine the target output directory.
SET_CEF_TARGET_OUT_DIR()
	
#include the subdirectory containing our libs
include_directories(${_CEF_ROOT})

#indicate the entry point for the executable
add_executable (cef_spout WIN32 ${ALL_SRCS} ${spout_src_files} ${spout_inc_files})
SET_EXECUTABLE_TARGET_PROPERTIES(cef_spout)

source_group("src" FILES ${ALL_SRCS})
source_group("spout" FILES ${spout_src_files} ${spout_inc_files})

add_dependencies(cef_spout libcef_dll_wrapper)

# Indicate which libraries to include during the link process.
target_link_libraries (cef_spout d3d11.lib Shlwapi.lib libcef_lib libcef_dll_wrapper opengl32)

COPY_FILES(cef_spout "${CEF_BINARY_FILES}" "${CEF_BINARY_DIR}" "${TARGET_OUTPUT_DIRECTORY}")
COPY_FILES(cef_spout "${CEF_RESOURCE_FILES}" "${CEF_RESOURCE_DIR}" "${TARGET_OUTPUT_DIRECTORY}")

# copy our test resources to output directory
set(mixer_resources overlay.png overlay.svg hud.html)
foreach(_resfile ${mixer_resources})
	add_custom_command(
			TARGET cef_spout POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy
					${CMAKE_SOURCE_DIR}/resource/${_resfile}
					${TARGET_OUTPUT_DIRECTORY}/resource/${_resfile})
endforeach()
  
if (MSVC)
    # remove default warning level from CMAKE_CXX_FLAGS_INIT
    string (REGEX REPLACE "/W[0-4]" "" CMAKE_CXX_FLAGS_INIT "${CMAKE_CXX_FLAGS_INIT}")
    string (REGEX REPLACE "/WX" "" CMAKE_CXX_FLAGS_INIT "${CMAKE_CXX_FLAGS_INIT}")
    target_compile_options(cef_spout PRIVATE "/W3" "/WX-")
endif()