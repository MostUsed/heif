# This file is part of Nokia HEIF library
#
# Copyright (c) 2015-2018 Nokia Corporation and/or its subsidiary(-ies). All rights reserved.
#
# Contact: heif@nokia.com
#
# This software, including documentation, is protected by copyright controlled by Nokia Corporation and/ or its subsidiaries. All rights are reserved.
#
# Copying, including reproducing, storing, adapting or translating, any or all of this material requires the prior written consent of Nokia.

cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

if(${CMAKE_EXTRA_GENERATOR} MATCHES "Eclipse CDT4")
    set(CMAKE_ECLIPSE_VERSION "4.4" CACHE STRING "Eclipse version" FORCE)
    set(CMAKE_CXX_COMPILER_ARG1 "-std=c++11" CACHE STRING "C++ version for eclipse" FORCE)
endif()

project(writer LANGUAGES CXX)

if(IOS)
    if(${IOS_PLATFORM} STREQUAL "OS")
        set(HEIF_WRITER_LIB_NAME heif_writer_static_iphoneos)
        set(HEIF_SHARED_WRITER_LIB_NAME heif_writer_shared_iphoneos)
    elseif(${IOS_PLATFORM} STREQUAL "SIMULATOR64")
        set(HEIF_WRITER_LIB_NAME heif_writer_static_iphonesimulator)
        set(HEIF_SHARED_WRITER_LIB_NAME heif_writer_shared_iphonesimulator)
    endif()
else()
    set(HEIF_WRITER_LIB_NAME heif_writer_static)
    set(HEIF_SHARED_WRITER_LIB_NAME heif_writer_shared)
endif()

set(WRITER_SRCS
    idgenerators.cpp
    refsgroup.cpp
    samplegroup.cpp
    timeutility.cpp
    writerimpl.cpp
    writermetaimpl.cpp
    writermoovimpl.cpp
    ../common/arraydatatype.cpp
    )


if(ANDROID)
  set(USE_LINUX_FILESTREAM 1)
else(ANDROID)
  set(USE_LINUX_FILESTREAM 0)
endif(ANDROID)

set(API_HDRS
    ../api/writer/heifwriter.h
    ../api/writer/heifwriterdatatypes.h
    ../api/common/heifallocator.h
    ../api/common/heifexport.h
    ../api/common/heifid.h
    )

set(WRITER_HDRS
    idgenerators.hpp
    refsgroup.hpp
    samplegroup.hpp
    timeutility.hpp
    writerconstants.hpp
    writerdatatypesinternal.hpp
    writerimpl.hpp
    )

macro(split_debug_info target)
  # objcopy doesn't work on MacOSX, so exclude that
  if(NOT CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
      # Use MATCHES for Clang to also include AppleClang.. though it's not really needed as it's excluded.
      add_custom_command(TARGET ${target}
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} --only-keep-debug $<TARGET_FILE:${target}> $<TARGET_FILE:${target}>.debug
    COMMAND ${CMAKE_OBJCOPY} --strip-debug $<TARGET_FILE:${target}>
    COMMAND ${CMAKE_OBJCOPY} --add-gnu-debuglink=$<TARGET_FILE:${target}>.debug $<TARGET_FILE:${target}>
    )
    endif()
  endif()
endmacro()

set(HEIF_WRITER_LIB_COMMON_DEFINES "HEIF_BUILDING_LIB" "_FILE_OFFSET_BITS=64" "_LARGEFILE64_SOURCE" "HEIF_WRITER_LIB")
if(USE_LINUX_FILESTREAM)
    set(HEIF_WRITER_LIB_COMMON_DEFINES ${HEIF_WRITER_LIB_COMMON_DEFINES} "HEIF_USE_LINUX_FILESTREAM")
endif()

add_library(${HEIF_WRITER_LIB_NAME} STATIC ${WRITER_SRCS} ${API_HDRS} ${WRITER_HDRS} $<TARGET_OBJECTS:common> )
set_property(TARGET ${HEIF_WRITER_LIB_NAME} PROPERTY CXX_STANDARD 11)
target_compile_definitions(${HEIF_WRITER_LIB_NAME} PRIVATE ${HEIF_WRITER_LIB_COMMON_DEFINES})
target_include_directories(${HEIF_WRITER_LIB_NAME} PRIVATE ../common)
target_include_directories(${HEIF_WRITER_LIB_NAME} PUBLIC ../api/common)
target_include_directories(${HEIF_WRITER_LIB_NAME} PUBLIC ../api/writer)
if (IOS)
    set_xcode_property(${HEIF_WRITER_LIB_NAME} IPHONEOS_DEPLOYMENT_TARGET "10.0")
endif(IOS)

if(NOT IOS)
    add_library(${HEIF_SHARED_WRITER_LIB_NAME} SHARED ${WRITER_SRCS} ${API_HDRS} ${WRITER_HDRS} $<TARGET_OBJECTS:common> )
    set_property(TARGET ${HEIF_SHARED_WRITER_LIB_NAME} PROPERTY CXX_STANDARD 11)
    if (CMAKE_BUILD_TYPE STREQUAL "Release" OR CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
      split_debug_info(${HEIF_SHARED_WRITER_LIB_NAME})
    endif()
    target_compile_definitions(${HEIF_SHARED_WRITER_LIB_NAME} PRIVATE "HEIF_BUILDING_DLL")
    target_compile_definitions(${HEIF_SHARED_WRITER_LIB_NAME} PRIVATE ${HEIF_WRITER_LIB_COMMON_DEFINES})
    target_include_directories(${HEIF_SHARED_WRITER_LIB_NAME} PRIVATE ../common)
    target_include_directories(${HEIF_SHARED_WRITER_LIB_NAME} PUBLIC ../api/common)
    target_include_directories(${HEIF_SHARED_WRITER_LIB_NAME} PUBLIC ../api/writer)
endif(NOT IOS)
