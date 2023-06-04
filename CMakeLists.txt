project(vigem_client CXX)
cmake_minimum_required(VERSION 3.20.2)

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
else()
	# Generate a static library, no link dependencies needed
	add_library(${PROJECT_NAME} STATIC ${SOURCES})
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
    EXPORT ${PROJECT_NAME}-export
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
install(EXPORT ${PROJECT_NAME}-export
	FILE ${PROJECT_NAME}Targets.cmake
	NAMESPACE ${PROJECT_NAME}::
	DESTINATION cmake
)
configure_file(ViGEmClient-config.cmake.in ViGEmClient-config.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/ViGEmClient-config.cmake DESTINATION cmake/)