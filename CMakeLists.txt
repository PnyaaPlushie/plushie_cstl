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
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

###############################################################################################
# Install logic
###############################################################################################

if(${PLUSHIE_CSTL_INCLUDE_INSTALL})
	include(CMakePackageConfigHelpers)
	include(GNUInstallDirs)
	
	# In local dev install mode, we don't install the source files, as that will
	# overwrite the developers source files. We also need to make the cmake 
	# install destination to be this current source file dir
	if(${PLUSHIE_CSTL_INSTALL_FOR_LOCAL_DEV})
		set(Plushie_INSTALL_CMAKEDIR ".")
	else()
		install(DIRECTORY include/plushie DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
		set(Plushie_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/plushie")
	endif()

	# Export the targets cmake
	
	install(TARGETS plushie_cstl EXPORT PlushieCStlTargets)
	install(EXPORT PlushieCStlTargets
		FILE PlushieCStlTargets.cmake 
		NAMESPACE Plushie:: 
		DESTINATION ${Plushie_INSTALL_CMAKEDIR})

	# Generate the versions file in the binary dir
	write_basic_package_version_file(
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfigVersion.cmake" 
		VERSION ${PlushieCstlVersion}
		COMPATIBILITY SameMajorVersion)

	# Use the .in template to generate the exported config
	configure_package_config_file(
		"${CMAKE_CURRENT_SOURCE_DIR}/cmake/PlushieCStlConfig.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		INSTALL_DESTINATION ${Plushie_INSTALL_CMAKEDIR}
		PATH_VARS CMAKE_INSTALL_INCLUDEDIR)

	# Install the config files
	install(FILES
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfig.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/PlushieCStlConfigVersion.cmake"
		DESTINATION ${Plushie_INSTALL_CMAKEDIR})

endif()