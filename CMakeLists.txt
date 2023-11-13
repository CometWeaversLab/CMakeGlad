# This CMake file provides a "target_create_glad()" and "target_add_dependency_glad(..)" functions
#  that can be used to add Glad as a dependency of one or more other CMake projects.
#
# See the "readme.md" file for usage example and details.
# 
# The "target_create_glad()" function will generate the Glad source files using the official
#  python script (located in the git submodule fog Glad that must be specified in the property "GLAD_GIT_SUBMODULE_PATH"). 
# A python 3.7+ interpreter must be installed on the system in order for said script to work.
# The parameters used by the python script to generate the Glad source files may be configured through 
# various CMake properties. This file defines a set of possible values for each aspect of the generation 
# of the Glad source files.
#
# The target_add_dependency_glad(DEPENDANT_TARGET_NAME) function setup the Glad target as a dependency
#  of the specified target. This function may be used on different target in case that more of them
#  requires to access Glad as a dependency. This function must be called after "target_create_glad()"
#  or it will generate an error.
#
# Author: Leonardo Spaccini

# If the Python interpreter is found, the "PYTHON_EXECUTABLE" property will provide its absolute path.
# The user is always capable of adjusting it in case of necessity.
find_package(PythonInterp 3.7 REQUIRED)

# OpenGL is not required for GLAD source file generation, but it is required for Glad to compile.
find_package(OpenGL REQUIRED)

set(GLAD_TARGTET_NAME "Glad" CACHE STRING "Display name for the Glad target")
set(GLAD_GIT_SUBMODULE_PATH "" CACHE STRING "Absolute path of the Git submodule for Glad")

# set the output directory for the generated source files of Glad as the "/dist" directory of the submodule, since it is already excluded from versioning by the ".gitignore" file.
set(GLAD_OUTPUT_PATH       "${GLAD_GIT_SUBMODULE_PATH}/dist" CACHE INTERNAL "")
set(GLAD_DIRECTORY_INCLUDE "${GLAD_OUTPUT_PATH}/include" CACHE INTERNAL "")
set(GLAD_DIRECTORY_SOURCE  "${GLAD_OUTPUT_PATH}/src" CACHE INTERNAL "")

# Glad options for the library mode
set(GLAD_LIBRARY_MODE "STATIC" CACHE STRING "Mode for the Glad library")
set_property(CACHE GLAD_LIBRARY_MODE PROPERTY STRINGS "HEADERS-ONLY" "STATIC" "SHARED")

# Glad options for the rendering API
set(GLAD_API_GL_PROFILE     "CORE" CACHE STRING "Profile of the OpenGL rendering API to be provided by Glad")
set(GLAD_API_GL_VERSION     "4.6"  CACHE STRING "Version of the OpenGL rendering API to be provided by Glad")
set(GLAD_API_GLES1_VERSION  "NONE" CACHE STRING "Version of the OpenGL ES (Embedded Systems) rendering API to be provided by Glad")
set(GLAD_API_GLES2_VERSION  "NONE" CACHE STRING "Version of the OpenGL ES2 (Embedded Systems) rendering API to be provided by Glad")
set(GLAD_API_GLSC2_VERSION  "NONE" CACHE STRING "Version of the OpenGL ES2 SC (Embedded Systems Safety-Critical) rendering API to be provided by Glad")
set(GLAD_API_VULKAN_VERSION "NONE" CACHE STRING "Version of the Vulkan API to be provided by Glad")

set_property(CACHE GLAD_API_GL_PROFILE     PROPERTY STRINGS "CORE" "COMPATIBILITY")
set_property(CACHE GLAD_API_GL_VERSION     PROPERTY STRINGS "NONE" "1.0" "1.1" "1.2" "1.3" "1.4" "1.5" "2.0" "2.1" "3.0" "3.1" "3.2" "3.3" "4.0" "4.1" "4.2" "4.3" "4.4" "4.5" "4.6")
set_property(CACHE GLAD_API_GLES1_VERSION  PROPERTY STRINGS "NONE" "1.0")
set_property(CACHE GLAD_API_GLES2_VERSION  PROPERTY STRINGS "NONE" "2.0" "3.0" "3.1" "3.2")
set_property(CACHE GLAD_API_GLSC2_VERSION  PROPERTY STRINGS "NONE" "2.0")
set_property(CACHE GLAD_API_VULKAN_VERSION PROPERTY STRINGS "NONE" "1.0" "1.1" "1.2")

# Glad options for the context creation API
set(GLAD_API_EGL_VERSION "NONE" CACHE STRING "Version of the EGL context creation API to be provided by Glad")
set(GLAD_API_GLX_VERSION "NONE" CACHE STRING "Version of the GLX (OpenGL extension for X11) context creation API to be provided by Glad")
set(GLAD_API_WGL_VERSION "NONE" CACHE STRING "Version of the WGL (OpenGL extension for Windows) context creation API to be provided by Glad")

set_property(CACHE GLAD_API_EGL_VERSION PROPERTY STRINGS "NONE" "1.0" "1.1" "1.2" "1.3" "1.4" "1.5")
set_property(CACHE GLAD_API_GLX_VERSION PROPERTY STRINGS "NONE" "1.0" "1.1" "1.2" "1.3" "1.4")
set_property(CACHE GLAD_API_WGL_VERSION PROPERTY STRINGS "NONE" "1.0")

# Glad options for the GL/EGL/WGL/GLX/VK API extensions
set(GLAD_API_EXTENSIONS "" CACHE STRING "List of GL/EGL/WGL/GLX/VK extensions API to be provided by Glad (comma-separated)")

# Additional Glad options
set(GLAD_OPT_ALIAS        OFF CACHE BOOL "Enables function pointer aliasing")
set(GLAD_OPT_DEBUG        OFF CACHE BOOL "Enable generation of a debug build")
set(GLAD_OPT_LOADER       OFF CACHE BOOL "Include internal loaders for APIs")
set(GLAD_OPT_MERGE        OFF CACHE BOOL "Merge multiple APIs of the same specification into one file")
set(GLAD_OPT_MX           OFF CACHE BOOL "Enables support for multiple GL contexts")
set(GLAD_OPT_MX_GLOBAL    OFF CACHE BOOL "Mimic global GL functions with context switching")
set(GLAD_OPT_ON_DEMAND    OFF CACHE BOOL "On-demand function pointer loading, initialize on use (experimental)")
set(GLAD_OPT_REPRODUCIBLE ON  CACHE BOOL "Makes the generation of Glad source files reproducible by not fetching the latest specification from Khronos")
set(GLAD_OPT_QUIET        ON  CACHE BOOL "Disable logging during generation of Glad source files")

function(target_create_glad)
	message(STATUS "Configuring target for Glad (taget name: \"${GLAD_TARGTET_NAME}\", git submodule path: \"${GLAD_GIT_SUBMODULE_PATH}/\")...")
	
	if(GLAD_GIT_SUBMODULE_PATH STREQUAL "")
		message(FATAL_ERROR "The property \"GLAD_SUBMODULE_FILES\" has not been set. Please, provide a value for this property before calling \"target_create_glad()\".")
	endif()
	if(NOT EXISTS "${GLAD_GIT_SUBMODULE_PATH}" OR NOT IS_DIRECTORY "${GLAD_GIT_SUBMODULE_PATH}")
		message(FATAL_ERROR "The value \"${GLAD_GIT_SUBMODULE_PATH}\" provided for property \"GLAD_SUBMODULE_FILES\" is not a valid path to a directory.")
	endif()
	
	if (TARGET ${GLAD_TARGTET_NAME})
	  	message(FATAL_ERROR "Could not create target for Glad (taget name: \"${GLAD_TARGTET_NAME}\") as it already exists! Did you call \"target_create_glad()\" multiple times?")
	endif()
	file(GLOB GLAD_SUBMODULE_FILES ${GLAD_GIT_SUBMODULE_PATH})
	list(LENGTH GLAD_SUBMODULE_FILES GLAD_SUBMODULE_FILES_COUNT)
	if(GLAD_SUBMODULE_FILES_COUNT EQUAL 0)
		message(FATAL_ERROR "The Glad submodule directory \"${GLAD_GIT_SUBMODULE_PATH}\" seems to be empty. Did you forgot to run \"git submodule update --init --recursive\"?")
	endif()
    
	# translate the GLAD options specified by the user into a command line string that will be used to generate the source files of the GLAD library.
	# this is done inside the function in order to allow the user to setup its properties before calling the function.
	if(NOT GLAD_API_GL_VERSION STREQUAL "NONE")
		string(TOLOWER "gl:${GLAD_API_GL_PROFILE}=${GLAD_API_GL_VERSION}" GLAD_TEMP)
		list(APPEND GLAD_CLI_API "${GLAD_TEMP}")
	endif()
	if(NOT GLAD_API_GLES1_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "gles1=${GLAD_API_GLES1_VERSION}")
	endif()
	if(NOT GLAD_API_GLES2_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "gles2=${GLAD_API_GLES2_VERSION}")
	endif()
	if(NOT GLAD_API_GLSC2_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "glsc2=${GLAD_API_GLSC2_VERSION}")
	endif()
	if(NOT GLAD_API_EGL_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "egl=${GLAD_API_GLSC2_VERSION}")
	endif()
	if(NOT GLAD_API_GLX_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "glx=${GLAD_API_GLX_VERSION}")
	endif()
	if(NOT GLAD_API_WGL_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "wgl=${GLAD_API_WGL_VERSION}")
	endif()
	if(NOT GLAD_API_VULKAN_VERSION STREQUAL "NONE")
		list(APPEND GLAD_CLI_API "vulkan=${GLAD_API_VULKAN_VERSION}")
	endif()

	if(GLAD_OPT_ALIAS)
		list(APPEND GLAD_CLI_OPTS "--alias")
	endif()
	if(GLAD_OPT_DEBUG)
		list(APPEND GLAD_CLI_OPTS "--debug")
	endif()
	if(GLAD_LIBRARY_MODE STREQUAL "HEADERS-ONLY")
		list(APPEND GLAD_CLI_OPTS "--header-only")
	endif()
	if(GLAD_OPT_LOADER)
		list(APPEND GLAD_CLI_OPTS "--loader")
	endif()
	if(GLAD_OPT_MX)
		list(APPEND GLAD_CLI_OPTS "--mx")
	endif()
	if(GLAD_OPT_MX_GLOBAL)
		list(APPEND GLAD_CLI_OPTS "--mx-global")
	endif()
	if(GLAD_OPT_ON_DEMAND)
		list(APPEND GLAD_CLI_OPTS "--on-demand")
	endif()

	if(GLAD_OPT_MERGE)
		set(GLAD_CLI_MERGE "--merge" CACHE INTERNAL "")
	endif()
	if(GLAD_OPT_REPRODUCIBLE)
		set(GLAD_CLI_REPRODUCIBLE "--reproducible" CACHE INTERNAL "")
	endif()
	if(GLAD_OPT_QUIET)
		set(GLAD_CLI_QUIET "--quiet" CACHE INTERNAL "")
	endif()

	list(JOIN GLAD_CLI_API "," GLAD_CLI_API)
	list(JOIN GLAD_CLI_OPTS " " GLAD_CLI_OPTS)
	set(GLAD_PYTHON_COMMAND_FULL "${PYTHON_EXECUTABLE} -m glad --api=${GLAD_CLI_API} --extensions=${GLAD_API_EXTENSIONS} ${GLAD_CLI_MERGE} --out-path ${GLAD_OUTPUT_PATH} ${GLAD_CLI_QUIET} ${GLAD_CLI_REPRODUCIBLE} c ${GLAD_CLI_OPTS}")

	# clean previous Glad source files (excluding this very "CMakeLists.txt" file).
	# FIXME: for some reason, emtpy directories are not removed. this is not a major issue but a detail that could be adjusted...
	message(STATUS "Cleaning previously generated Glad source files (if any) from \"${GLAD_OUTPUT_PATH}/\"...")
	file(GLOB_RECURSE GLAD_FILES_TO_CLEAN "${GLAD_OUTPUT_PATH}/*.*")
	foreach(F ${GLAD_FILES_TO_CLEAN})
	  if(${F} MATCHES "CMakeLists.txt")
		list(REMOVE_ITEM GLAD_FILES_TO_CLEAN ${F})
	  endif()
	endforeach()
	if(GLAD_FILES_TO_CLEAN)
		file(REMOVE_RECURSE ${GLAD_FILES_TO_CLEAN})
	endif()
	
	# generate the source files of Glad through the official python script.
	# also, print the current working directory (CWD) and the python command (CMD) that are executed.
	message(STATUS "Generating Glad source files (CWD: \"${GLAD_GIT_SUBMODULE_PATH}/\", CMD:\"${GLAD_PYTHON_COMMAND_FULL}\")...")
	execute_process(COMMAND cmd /c "${GLAD_PYTHON_COMMAND_FULL}"
		WORKING_DIRECTORY "${GLAD_GIT_SUBMODULE_PATH}"
		COMMAND_ERROR_IS_FATAL ANY
	)

	# collect the source files of Glad.
	file(GLOB_RECURSE GLAD_FILES_INCLUDE "${GLAD_DIRECTORY_INCLUDE}/*.*")
	file(GLOB_RECURSE GLAD_FILES_SOURCE "${GLAD_DIRECTORY_SOURCE}/*.*")
	
	# add Glad as library depending on the configured library mode.
	if(GLAD_LIBRARY_MODE STREQUAL "HEADERS-ONLY")
		message(STATUS "Configuring Glad (taget name: \"${GLAD_TARGTET_NAME}\") as an header-only library...")
		add_library(${GLAD_TARGTET_NAME} INTERFACE ${GLAD_FILES_INCLUDE})
    elseif(GLAD_LIBRARY_MODE STREQUAL "STATIC")
		message(STATUS "Configuring Glad (taget name: \"${GLAD_TARGTET_NAME}\") as a static library...")
		add_library(${GLAD_TARGTET_NAME} STATIC ${GLAD_FILES_INCLUDE} ${GLAD_FILES_SOURCE})
		target_include_directories(${GLAD_TARGTET_NAME} PUBLIC "${GLAD_DIRECTORY_INCLUDE}")
		set_target_properties(${GLAD_TARGTET_NAME} PROPERTIES OUTPUT_NAME "glad")
	elseif(GLAD_LIBRARY_MODE STREQUAL "SHARED")
		message(STATUS "Configuring Glad (taget name: \"${GLAD_TARGTET_NAME}\") as a shared library...")
		add_compile_definitions(GLAD_API_CALL_EXPORT)
		add_library(${GLAD_TARGTET_NAME} SHARED ${GLAD_FILES_INCLUDE} ${GLAD_FILES_SOURCE})
		target_include_directories(${GLAD_TARGTET_NAME} PUBLIC "${GLAD_DIRECTORY_INCLUDE}")
		target_link_libraries(${GLAD_TARGTET_NAME} "${OPENGL_LIBRARIES}")
		set_target_properties(${GLAD_TARGTET_NAME} PROPERTIES OUTPUT_NAME "glad")
	endif()

	# add groups preserving the directory tree structure (Visual Studio)
	source_group(TREE "${GLAD_DIRECTORY_INCLUDE}" PREFIX "Include" FILES ${GLAD_FILES_INCLUDE})
	source_group(TREE "${GLAD_DIRECTORY_SOURCE}" PREFIX "Source" FILES ${GLAD_FILES_SOURCE})
endfunction()

function(target_add_dependency_glad DEPENDANT_TARGET_NAME)
	message(STATUS "Adding Glad (taget name: \"${GLAD_TARGTET_NAME}\") as a dependency of target \"${DEPENDANT_TARGET_NAME}\"...")
	if (NOT TARGET ${GLAD_TARGTET_NAME})
	  	message(FATAL_ERROR "Could not add target Glad (taget name: \"${GLAD_TARGTET_NAME}\") as a dependency because it does not exists! (Did you forgot to call \"target_create_glad()\" ?)")
	endif()
	
	# add the Glad include directory as a private include dependency to the dependant target
	target_include_directories(${DEPENDANT_TARGET_NAME} PRIVATE "${GLAD_DIRECTORY_INCLUDE}")
	
	# in case static/shared library mode is configured for Glad,
	# must also link the static library to the dependant target.
	if(NOT GLAD_LIBRARY_MODE STREQUAL "HEADERS-ONLY")
		target_link_libraries(${DEPENDANT_TARGET_NAME} ${GLAD_TARGTET_NAME})
	endif()
endfunction()