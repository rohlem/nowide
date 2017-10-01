include(GNUInstallDirs)

option(NOWIDE_USE_BOOST_FILESYSTEM "Build with support for boost filesystem. Ignored for standalone version")

option(NOWIDE_BUILD_SHARED "Build shared library" ON)
option(NOWIDE_BUILD_STATIC "Build static library" ON)

# Get path to nowide
get_filename_component(NOWIDE_DIR ${CMAKE_CURRENT_SOURCE_DIR} DIRECTORY)
# Find out if we are beeing build as standalone or boost version
if(EXISTS ${NOWIDE_DIR}/include/nowide)
	set(NOWIDE_STANDALONE ON CACHE INTERNAL "Build without boost")
	set(NOWIDE_HEADER_DIR ${NOWIDE_DIR}/include/nowide)
else()
	set(NOWIDE_STANDALONE OFF CACHE INTERNAL "Build with boost")
	set(NOWIDE_HEADER_DIR ${NOWIDE_DIR}/include/boost/nowide)
endif()

# Using glob here is ok as it is only for headers
file(GLOB NOWIDE_HEADERS ${NOWIDE_HEADER_DIR}/*.hpp)

if(NOT NOWIDE_STANDALONE)
	# Default boost libs are static on self-build msvc versions and dynamic in the linux package repos
	if(MSVC AND "${Boost_USE_STATIC_LIBS}" STREQUAL "")
		set(Boost_USE_STATIC_LIBS ON)
	endif()

	find_package(Boost 1.55 REQUIRED COMPONENTS Locale)
	if(NOWIDE_USE_BOOST_FILESYSTEM)
		find_package(Boost 1.55 REQUIRED COMPONENTS Filesystem System)
	endif()
	# Can be replaced in CMake 2.8.12
	include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
endif()

# Can be replaced in CMake 2.8.12
include_directories(${NOWIDE_DIR}/include)

if(NOWIDE_BUILD_SHARED)
	add_library(nowide SHARED iostream.cpp ${NOWIDE_HEADERS})
	set_target_properties(nowide PROPERTIES VERSION 0.0.0 SOVERSION 0)

	# Not required for CMake 2.8
	# set_target_properties(nowide PROPERTIES CLEAN_DIRECT_OUTPUT 1)

	# Requires CMake 2.8.12
	# target_include_directories(nowide PUBLIC ${NOWIDE_DIR}/include)
	if(NOWIDE_STANDALONE)
		set_property(TARGET nowide PROPERTY COMPILE_DEFINITIONS NOWIDE_EXPORT NOWIDE_SOURCE)
		# Better in CMake 2.8.12
		# target_compile_definitions(nowide PUBLIC NOWIDE_EXPORT PRIVATE NOWIDE_SOURCE)
	else()
		set_property(TARGET nowide PROPERTY COMPILE_DEFINITIONS BOOST_NOWIDE_DYN_LINK NOWIDE_SOURCE)
		# Better in CMake 2.8.12
		# target_compile_definitions(nowide PUBLIC BOOST_NOWIDE_DYN_LINK BOOST_NOWIDE_NO_LIB PRIVATE BOOST_NOWIDE_SOURCE)
		# target_include_directories(nowide SYSTEM PUBLIC ${Boost_INCLUDE_DIRS})
		# In CMake 2.8.12 replace LINK_INTERFACE_LIBRARIES by INTERFACE and remove policy CMP0022
		target_link_libraries(nowide LINK_INTERFACE_LIBRARIES ${Boost_LOCALE_LIBRARY})
		if(NOWIDE_USE_BOOST_FILESYSTEM)
		# Requires CMake 2.8.12
		#	target_compile_definitions(nowide PUBLIC BOOST_NOWIDE_USE_FILESYSTEM)
			target_link_libraries(nowide LINK_INTERFACE_LIBRARIES ${Boost_FILESYSTEM_LIBRARY} ${Boost_SYSTEM_LIBRARY})
		endif()
	endif()

	install(TARGETS nowide
			RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
			LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
			ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

if(NOWIDE_BUILD_STATIC)
	add_library(nowide-static STATIC iostream.cpp ${NOWIDE_HEADERS})
	# Rename to libnowide and enable linking into shared libs
	set_target_properties(nowide-static PROPERTIES OUTPUT_NAME "nowide" PREFIX "lib" POSITION_INDEPENDENT_CODE ON)

	# Not required for CMake 2.8
	# set_target_properties(nowide-static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

	# Requires CMake 2.8.12
	# target_include_directories(nowide-static PUBLIC ${NOWIDE_DIR}/include)
	if(NOT NOWIDE_STANDALONE)
		# Requires CMake 2.8.12
		# target_include_directories(nowide-static SYSTEM PUBLIC ${Boost_INCLUDE_DIRS})
		# target_compile_definitions(nowide-static PUBLIC BOOST_NOWIDE_NO_LIB)
		target_link_libraries(nowide-static ${Boost_LOCALE_LIBRARY})
		if(NOWIDE_USE_BOOST_FILESYSTEM)
		# Requires CMake 2.8.12
		#	target_compile_definitions(nowide-static PUBLIC BOOST_NOWIDE_USE_FILESYSTEM)
			target_link_libraries(nowide-static ${Boost_FILESYSTEM_LIBRARY} ${Boost_SYSTEM_LIBRARY})
		endif()
	endif()
	install(TARGETS nowide-static
			RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
			LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
			ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

install(DIRECTORY ${NOWIDE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})