
option(NOWIDE_RUN_WITH_WINE "Use wine to run tests" OFF)

# Executables linking to nowide get their include paths automatically
add_executable(test_iostream_shared test_iostream.cpp)
target_link_libraries(test_iostream_shared nowide)

add_executable(test_iostream_static test_iostream.cpp)
target_link_libraries(test_iostream_static nowide-static)

# The others need it explicitely (until CMake 3.0.0 which could provide an INTERFACE library to "link" against)
# Note: One could link against nowide-static which is empty on non-windows builds
include_directories(${NOWIDE_INCLUDE_DIRS})
if(NOT NOWIDE_STANDALONE)
	find_package(Boost 1.55 REQUIRED)
	include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
endif()

set(NOWIDE_TESTS
    test_convert
    test_stdio
    test_fstream
    )

foreach(TEST ${NOWIDE_TESTS})
    add_executable(${TEST} ${TEST}.cpp)
    if(NOWIDE_RUN_WITH_WINE)
       add_test(NAME ${TEST} COMMAND wine $<TARGET_FILE:${TEST}>)
    else()
       add_test(NAME ${TEST} COMMAND ${TEST})
    endif()
endforeach()

add_executable(test_system test_system.cpp)
add_executable(test_env_proto test_env.cpp)
add_executable(test_env_win test_env.cpp)
set_target_properties(test_env_win PROPERTIES COMPILE_DEFINITIONS NOWIDE_TEST_INCLUDE_WINDOWS)

set(NOWIDE_OTHER_TESTS test_iostream_shared test_iostream_static test_env_win test_env_proto)
if(NOT NOWIDE_STANDALONE AND NOWIDE_USE_BOOST_FILESYSTEM)
    add_executable(test_fs test_fs.cpp)
    target_link_libraries(test_fs nowide-static) # Not acutally required but pulls in boost deps and compile definitions
    list(APPEND OTHER_TESTS test_fs)
endif()

if(NOWIDE_RUN_WITH_WINE)
	foreach(TEST ${NOWIDE_OTHER_TESTS})
		add_test(NAME ${TEST} COMMAND wine $<TARGET_FILE:${TEST}>)
	endforeach()

	add_test(NAME test_system_n COMMAND wine $<TARGET_FILE:test_system> "-n")
	add_test(NAME test_system_w COMMAND wine $<TARGET_FILE:test_system> "-w")
else()
	foreach(TEST ${NOWIDE_OTHER_TESTS})
		add_test(NAME ${TEST} COMMAND ${TEST})
	endforeach()

	add_test(NAME test_system_n COMMAND test_system "-n")
	add_test(NAME test_system_w COMMAND test_system "-w")
endif()
