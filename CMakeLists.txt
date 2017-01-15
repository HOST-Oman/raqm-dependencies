cmake_minimum_required(VERSION 2.8.12)
include(CMakeDependentOption)

if (NOT APPLE AND NOT WIN32 AND NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
message(FATAL_ERROR
"This dependency pack is intended for use on Apple OS X and MS Windows only. "
"For other systems, such as GNU/Linux, please consult your package manager or "
"distribution maintainer to discover how to install the libraries you wish to "
"use with RAQM. Thanks!")
endif()

if (APPLE)
    set( CMAKE_OSX_ARCHITECTURES x86_64 CACHE STRING "Build architectures for Mac OS X" )
    set( CMAKE_BUILD_TYPE Release CACHE STRING "Sets the configuration to build (Release, Debug, etc...)")
endif()

project(RAQM-DEPS)

list( APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" )
include( RAQM-deps-macros )

set(CMAKE_DEBUG_POSTFIX _d)

cmake_dependent_option( RAQM_MSVC_STATIC_RUNTIME "Specifies whether to the static runtime (/MT and /MTd) or the DLL runtime (/MD and /MDd)." FALSE "MSVC" FALSE )
mark_as_advanced(RAQM_MSVC_STATIC_RUNTIME)

if (APPLE)
    set(RAQM_APPLE_DYLIB_INSTALL_PATH "@executable_path/../Frameworks" CACHE STRING "Specifies the install path for generated dylibs." )
    mark_as_advanced(RAQM_APPLE_DYLIB_INSTALL_PATH)
endif()

cegui_check_mingw()
cegui_check_msvc()

if (RAQM_MSVC_STATIC_RUNTIME)
    foreach(_BUILD_CONFIG DEBUG RELEASE RELWITHDEBINFO MINSIZEREL)
        foreach(_BUILD_VARS CMAKE_CXX_FLAGS_${_BUILD_CONFIG} CMAKE_C_FLAGS_${_BUILD_CONFIG})
            string(REGEX REPLACE /MD /MT ${_BUILD_VARS} ${${_BUILD_VARS}})
        endforeach()
    endforeach()
endif()

if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

macro(raqm_add_library _NAME _SOURCES_VAR _HEADERS_VAR)
    add_library(${_NAME} SHARED ${${_SOURCES_VAR}} ${${_HEADERS_VAR}})

    set_target_properties(${_NAME} PROPERTIES
        ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"

        LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        LIBRARY_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"
        LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/lib/dynamic"

        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/bin")

    if (APPLE)
        set_target_properties(${_NAME} PROPERTIES
            INSTALL_NAME_DIR ${RAQM_APPLE_DYLIB_INSTALL_PATH}
            BUILD_WITH_INSTALL_RPATH TRUE
        )
    endif()

    add_library(${_NAME}_static STATIC ${${_SOURCES_VAR}} ${${_HEADERS_VAR}})

    set_target_properties(${_NAME}_static PROPERTIES
        ARCHIVE_OUTPUT_NAME ${_NAME}
        LIBRARY_OUTPUT_NAME ${_NAME}
        RUNTIME_OUTPUT_NAME ${_NAME}

        ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/lib/static"

        LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        LIBRARY_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/lib/static"
        LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/lib/static"

        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/dependencies/bin"
        RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/dependencies/bin")

endmacro()

set(RAQM_BUILD_HARFBUZZ TRUE)
set(RAQM_BUILD_FRIBIDI TRUE)
set(RAQM_BUILD_FREETYPE2 TRUE)

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/dependencies/include")



if (RAQM_BUILD_FREETYPE2)
    add_subdirectory(src/freetype-2.7.1/RAQM-BUILD)
    set(freetypeDep freetype)
endif()

if (RAQM_BUILD_FRIBIDI)
    add_subdirectory(src/fribidi-0.19.7/RAQM-BUILD)
    set(fribidiDep fribidi)
endif()

if (RAQM_BUILD_HARFBUZZ)
    add_subdirectory(src/harfbuzz-1.2.7/RAQM-BUILD)
    set(harfbuzzDep harfbuzz)
endif()


add_custom_target(POST_BUILD_CLEANUP ALL DEPENDS
    ${fribidiDep} ${harfbuzzDep} ${freetypeDep})

if (MSVC)
    add_custom_command(TARGET POST_BUILD_CLEANUP COMMAND ${CMAKE_COMMAND} -P "${CMAKE_SOURCE_DIR}/msvc_postbuild.cmake")
endif()
