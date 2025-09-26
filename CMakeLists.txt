cmake_minimum_required(VERSION 3.0)
project(UnitTest++ VERSION 2.1.0)

option(UTPP_USE_PLUS_SIGN
    "Set this to OFF if you wish to use '-cpp' instead of '++' in lib/include paths"
    ON)
option(UTPP_INCLUDE_TESTS_IN_BUILD
    "Set this to OFF if you do not wish to automatically build or run unit tests as part of the default cmake --build"
    ON)

# get the main sources
file(GLOB headers_ RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} UnitTest++/*.h)
file(GLOB sources_ RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} UnitTest++/*.cpp)
source_group("" FILES ${headers_} ${sources_})

# get platform specific sources
if (WIN32)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    set(platformDir_ Win32)
else()
    set(platformDir_ Posix)
endif(WIN32)

file(GLOB platformHeaders_ RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} UnitTest++/${platformDir_}/*.h)
file(GLOB platformSources_ RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} UnitTest++/${platformDir_}/*.cpp)
source_group(${platformDir_} FILES ${platformHeaders_} ${platformSources_})

# create the lib
add_library(UnitTest++ STATIC ${headers_} ${sources_} ${platformHeaders_} ${platformSources_})
add_library(UnitTest++::UnitTest++ ALIAS UnitTest++)


if(${UTPP_USE_PLUS_SIGN})
	set_target_properties(UnitTest++ PROPERTIES OUTPUT_NAME UnitTest++)
endif()


# build the test runner
file(GLOB TEST_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} tests/*.cpp tests/*.h)
source_group( "" FILES ${TEST_SRCS})
add_executable(TestUnitTest++ ${TEST_SRCS})


if(${UTPP_USE_PLUS_SIGN})
	set_target_properties(TestUnitTest++ PROPERTIES OUTPUT_NAME TestUnitTest++)
endif()

target_link_libraries(TestUnitTest++ 
	PUBLIC
		UnitTest++::UnitTest++
	)

# run unit tests as post build step
add_custom_command(TARGET TestUnitTest++
    POST_BUILD COMMAND TestUnitTest++
    COMMENT "Running unit tests")

if(NOT ${UTPP_INCLUDE_TESTS_IN_BUILD})
    set_target_properties(TestUnitTest++ PROPERTIES EXCLUDE_FROM_ALL 1)
endif()


target_include_directories( UnitTest++
	PUBLIC
		$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include/UnitTest++>
	) 
set_target_properties(UnitTest++ PROPERTIES DEBUG_POSTFIX "-d")
set_target_properties(TestUnitTest++ PROPERTIES DEBUG_POSTFIX "-d")
