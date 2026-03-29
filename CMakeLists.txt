###############################################################################################
# Plushie C STL
# Entry CMakeLists file for header only additional c libs that I reuse
###############################################################################################
cmake_minimum_required(VERSION 3.16.0 FATAL_ERROR)
option(PLUSHIE_CSTL_INCLUDE_INSTALL "Include the install target for Plushie Cstl" ON)
option(PLUSHIE_CSTL_INSTALL_FOR_LOCAL_DEV "Generate find_package to use dev source files" OFF)

set(PlushieCstlVersion 0.1.0)
project(PlushieCStl VERSION ${PlushieCstlVersion} LANGUAGES CXX C)

# Add the interface library
add_library(plushie_cstl INTERFACE)
add_library(Plushie::CStl ALIAS plushie_cstl)

set_target_properties(plushie_cstl PROPERTIES 
	EXPORT_NAME CStl)

target_include_directories(plushie_cstl INTERFACE 
	# Used with `add_subdirectory` or `FetchContent` 
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>

	# Used with find_package from install 
    $<INSTALL_INTERFACE:include>) 

###############################################################################################
# Install logic
###############################################################################################

# Install is the last bit of the file
if(NOT ${PLUSHIE_CSTL_INCLUDE_INSTALL})
	return()
endif()

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
	
# Create an export set for Plushie::CStl 
install(TARGETS plushie_cstl EXPORT PlushieCStlTargets)

if(NOT ${PLUSHIE_CSTL_INSTALL_FOR_LOCAL_DEV})
	# Performing a regular portable install, this includes copying all of the 
	# header files, and generating the regular packaging files

	# Copy all header files in the include directory 
	install(DIRECTORY include/plushie DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

	# Relocatable targets file 
	install(EXPORT PlushieCStlTargets
		NAMESPACE Plushie::
		FILE PlushieCStlTargets.cmake 
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/plushie)

	# Use the .in template to generate the exported config
	configure_package_config_file(
		"${CMAKE_CURRENT_SOURCE_DIR}/cmake/PlushieCStlConfig.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/plushie
		PATH_VARS CMAKE_INSTALL_INCLUDEDIR)

	# Actually export the config file
	install(FILES
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/plushie)

else()
	# Performing a dev local install, skips copying any files, it just aims to 
	# place the packaging config file and point them at this source dir

	# Use regular configure_file as we're making a "good enough" fake config
	set(_plushie_cstl_root ${CMAKE_CURRENT_LIST_DIR})
	configure_file(
		"${CMAKE_CURRENT_SOURCE_DIR}/cmake/PlushieCStlConfig_LocalDev.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		@ONLY)

	# Actually export the config file
	install(FILES
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		DESTINATION ${CMAKE_CURRENT_LIST_DIR})
endif()


