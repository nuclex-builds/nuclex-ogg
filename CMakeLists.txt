#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  Ogg
  VERSION 1.3.5
  DESCRIPTION "Media container for Opus, Vorbis (audio) and Theora (video) streams"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/xiph/ogg/releases/download/v1.3.5/libogg-1.3.5.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/libogg-1.3.5.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=0eb4b4b9420a0f51db142ba3f9c64b333f826532dc0f48c6410ae51f4799b664
    STATUS DOWNLOAD_STATUS
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    list(GET DOWNLOAD_STATUS 2 HTTP_CODE)
    message(
      FATAL_ERROR
      "Could not download OGG sources: ${ERROR_MESSAGE} (HTTP result ${HTTP_CODE})"
    )
  endif()

  message(STATUS "Extracting OGG sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/libogg-1.3.5.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/libogg-1.3.5
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

#configure_file(
#	"${PROJECT_SOURCE_DIR}/config.h"
#	"${PROJECT_SOURCE_DIR}/build/config.h"
#	COPYONLY
#)

# -------------------------------------------------------------------------------------------------

set(
  headerFiles
  "build/include/ogg/os_types.h"
  "build/include/ogg/ogg.h"
  #"build/include/ogg/config_types.h.in"
)
set(
  sourceFiles
  "build/src/bitwise.c"
  "build/src/framing.c"
  #"build/src/crctable.h"
)

# -------------------------------------------------------------------------------------------------

add_library(Ogg STATIC)
add_library(Ogg::Static ALIAS Ogg)

#target_compile_definitions(
#  Ogg
#  PRIVATE OGG_BUILD
#)

target_include_directories(
  Ogg
  PUBLIC "build/include"
)

target_sources(
  Ogg
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(Ogg PROPERTIES PREFIX "")
set_target_properties(Ogg PROPERTIES OUTPUT_NAME "ogg")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS Ogg
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
