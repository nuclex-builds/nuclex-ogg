#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  Ogg
  VERSION 1.3.5
  DESCRIPTION "Media container for Ogg, Vorbis (audio) and Theora (video) streams"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

# TODO: Effin' CMake again. Can't this abomination of a useless build system just die?
#  If downloads the file. It complains about the checksum if it's wrong.
# Then it writes a BAD status into DOWNLOAD_STATUS. And leaves ERROR_MESSAGE empty.
# Completely useless. Weird, custom build language with side effects as the primary
# means of returning results and that probably changes between versions. Not fixing this.
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/xiph/ogg/releases/download/v1.3.5/libogg-1.3.5.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/libogg-1.3.5.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=7875d49596286055b52439ed42f044bd8ad426aa4cc5aabd96bfe7abb971d5e8
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/libogg-1.3.5.tar.gz)
    message(FATAL_ERROR "Could not download Ogg: ${ERROR_MESSAGE}")
  endif()

  message(STATUS "Extracting Ogg sources")
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
  sourceFiles
  "build/src/bitwise.c"
  "build/src/framing.c"
  #"build/src/crctable.h"
)
set(
  headerFiles
  "build/include/ogg/os_types.h"
  "build/include/ogg/ogg.h"
  #"build/include/ogg/config_types.h.in"
)

# -------------------------------------------------------------------------------------------------

add_library(Ogg STATIC)
add_library(Ogg::Static ALIAS Ogg)

target_compile_definitions(
  Ogg
  PRIVATE OGG_BUILD
)

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

# -------------------------------------------------------------------------------------------------

file(
  WRITE "${PROJECT_SOURCE_DIR}/OggConfig.cmake"
  "#!/usr/bin/cmake

# Configuration to include Ogg in a CMake-based project. If you want to
# reference Ogg as an externally compiled static library, do this:
#
#    set(Ogg_DIR \"../ThirdParty/ogg\")
#    find_package(Ogg REQUIRED CONFIG)
#
#    target_link_libraries(
#      MyAwesomeProject
#      PRIVATE Ogg::Static
#    )
#
# Alternatively, if you want to build Ogg together with your project,
# use the normal CMakeLists.txt with CMake's add_subdirectory() command:
#
#    add_subdirectory(
#      \"\${PROJECT_SOURCE_DIR}/../ThirdParty/ogg\"
#      \"\${CMAKE_BINARY_DIR}/ogg\"
#    )
#
#    target_link_libraries(
#      MyAwesomeProject
#      PRIVATE Ogg
#    )
#

# -------------------------------------------------------------------------------------------------

if(NOT DEFINED NUCLEX_COMPILER_TAG)
  message(
    FATAL_ERROR
    \"NUCLEX_COMPILER_TAG not defined! Include cplusplus.cmake before importing this package \\
    in order to generate a tag identifying the platform/compiler/architecture/variant!\"
  )
endif()

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}\")
  # TODO: Warn and link release build when compiling in debug mode
  # TODO: Warn and link build for older compiler version if found
  message(
    FATAL_ERROR
    \"Directory '\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}' not found. \\
    Please either build and install this project before importing it via \\
    find_package() or use this project's main CMakeFiles.txt via add_subdirectory()!\"
  )
endif()

# -------------------------------------------------------------------------------------------------

add_library(Ogg::Static STATIC IMPORTED)

# This may cause warnings on recent GCC versions (10.0.0+?) with LTO because GCC detects
# that the headers used during build (residing in build/) are not the same used when
# linking the library (copies resising in Include/).
#
# CMake doesn't run the install step during build, so the only way to get the headers
# in place before building would be by copying them rather than installing them.
set_target_properties(
  Ogg::Static PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES \"\${CMAKE_CURRENT_LIST_DIR}/Include\"
  IMPORTED_LINK_INTERFACE_LANGUAGES \"C\"
)

if(WIN32)
  set_target_properties(
    Ogg::Static PROPERTIES
    IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/ogg.lib\"
  )
else()
  set_target_properties(
    Ogg::Static PROPERTIES
    IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/ogg.a\"
  )
endif()

message(STATUS \"Imported Ogg targets with binaries in '\${CMAKE_CURRENT_LIST_DIR}'\")")

# -------------------------------------------------------------------------------------------------
