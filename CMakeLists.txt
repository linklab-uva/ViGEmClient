set(_required_cmake 3.15)
cmake_minimum_required (VERSION ${_required_cmake} FATAL_ERROR)
cmake_policy(VERSION ${_required_cmake})
project(vigem_client CXX)

find_package(ament_cmake QUIET)
if(NOT MSVC)
	message(STATUS "vigem_client only supported on Windows. Not building")
	if(ament_cmake_FOUND)
		ament_package()
	endif()
	return()
endif()
# use -DViGEmClient_DLL=ON on the cmake command line to change this value
option(ViGEmClient_DLL "Generate a dynamic library instead of a static library" OFF)

set(SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/ViGEmClient.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/Internal.h ${CMAKE_CURRENT_SOURCE_DIR}/src/resource.h ${CMAKE_CURRENT_SOURCE_DIR}/src/ViGEmClient.rc)
if(ViGEmClient_DLL)
	# Generate a dynamic library with proper link dependencies
	add_library(${PROJECT_NAME} SHARED ${SOURCES})
	target_link_libraries (${PROJECT_NAME} PUBLIC setupAPI.lib)
	target_compile_definitions(${PROJECT_NAME} 
		PUBLIC "VIGEM_DYNAMIC"
		PRIVATE "VIGEM_EXPORTS"
	)
	set(VIGEM_DYNAMIC True)
else()
	# Generate a static library, no link dependencies needed
	add_library(${PROJECT_NAME} STATIC ${SOURCES})
	set(VIGEM_DYNAMIC False)
endif()

target_include_directories(${PROJECT_NAME} 
	PUBLIC
		$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
		$<INSTALL_INTERFACE:include>
)

install(DIRECTORY include/ViGEm
	DESTINATION include
	FILES_MATCHING PATTERN "*.h"
)
install(TARGETS ${PROJECT_NAME}
    EXPORT export_${PROJECT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
install(EXPORT export_${PROJECT_NAME}
	FILE ${PROJECT_NAME}Targets.cmake
	NAMESPACE ${PROJECT_NAME}::
	DESTINATION cmake
)
if(${ament_cmake_FOUND})
	ament_export_targets(export_${PROJECT_NAME})
	ament_package(CONFIG_EXTRAS vigem_client_cmake_extras.cmake.in)
else()
	configure_file(ViGEmClient-config.cmake.in vigem_client-config.cmake @ONLY)
	install(FILES ${CMAKE_BINARY_DIR}/vigem_client-config.cmake DESTINATION cmake/)
endif()