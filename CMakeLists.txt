cmake_minimum_required(VERSION 3.8.0)

if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "No build type selected, default to Release")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release" FORCE)
endif()

project(giflib VERSION 5.2.1 LANGUAGES C)

# configuration options
option(GIF_ENABLE_STATIC "Build static giflib library." ON)
option(GIF_ENABLE_SHARED "Build shared giflib library." OFF)
if(NOT GIF_ENABLE_STATIC AND NOT GIF_ENABLE_SHARED)
    set(GIF_ENABLE_STATIC ON)
endif()

add_library(gif_object_lib OBJECT
    dgif_lib.c
    egif_lib.c
    gifalloc.c
    gif_err.c
    gif_font.c
    gif_hash.c
    openbsd-reallocarray.c
    gif_hash.h
    gif_lib.h
    gif_lib_private.h
)

set_target_properties(gif_object_lib PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_compile_features(gif_object_lib PRIVATE c_std_99)
target_compile_definitions(gif_object_lib PRIVATE _CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_DEPRECATE)

if(GIF_ENABLE_STATIC)
    add_library(gif_static_lib STATIC $<TARGET_OBJECTS:gif_object_lib>)
    set_target_properties(gif_static_lib PROPERTIES PUBLIC_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/gif_lib.h OUTPUT_NAME gif)
    target_include_directories(gif_static_lib PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
    )
endif()

if(GIF_ENABLE_SHARED)
    add_library(gif_shared_lib SHARED $<TARGET_OBJECTS:gif_object_lib>)
    set_target_properties(gif_shared_lib PROPERTIES 
        OUTPUT_NAME gif
        POSITION_INDEPENDENT_CODE ON
        PUBLIC_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/gif_lib.h
        SOVERSION 7.2.0
        VERSION 5.2.1)

    set_target_properties(gif_shared_lib PROPERTIES ARCHIVE_OUTPUT_NAME gif_import)

    target_include_directories(gif_shared_lib PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
    )
endif()
