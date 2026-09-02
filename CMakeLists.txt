#  MinHook - The Minimalistic API Hooking Library for x64/x86
#  Copyright (C) 2009-2017 Tsuda Kageyu.
#  All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions
#  are met:
#
#   1. Redistributions of source code must retain the above copyright
#      notice, this list of conditions and the following disclaimer.
#   2. Redistributions in binary form must reproduce the above copyright
#      notice, this list of conditions and the following disclaimer in the
#      documentation and/or other materials provided with the distribution.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
#  "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
#  TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
#  PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER
#  OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
#  EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
#  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
#  PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
#  LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
#  NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
#  SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 4.4.3)

project(minhook
  VERSION 1.3.3
  DESCRIPTION "The Minimalistic x86/x64 API Hooking Library for Windows"
  HOMEPAGE_URL "https://github.com/TsudaKageyu/minhook"
  LANGUAGES C
)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

set(MINHOOK_MAJOR_VERSION ${PROJECT_VERSION_MAJOR})
set(MINHOOK_MINOR_VERSION ${PROJECT_VERSION_MINOR})
set(MINHOOK_PATCH_VERSION ${PROJECT_VERSION_PATCH})
set(MINHOOK_VERSION ${PROJECT_VERSION})

################
#    BUILD     #
################

option(BUILD_SHARED_LIBS "build shared version" OFF)

set(SOURCES_MINHOOK
  "src/buffer.c"
  "src/hook.c"
  "src/trampoline.c"
)

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(SOURCES_HDE "src/hde/hde64.c")
else()
  set(SOURCES_HDE "src/hde/hde32.c")
endif()

set(RESOURCES "")

if(WIN32 AND BUILD_SHARED_LIBS)
  enable_language(RC)
  set(RESOURCES
    "dll_resources/MinHook.rc"
    "dll_resources/MinHook.def"
  )
endif()

add_library(minhook ${SOURCES_MINHOOK} ${SOURCES_HDE} ${RESOURCES})
add_library(minhook::minhook ALIAS minhook)

target_sources(minhook PUBLIC
  FILE_SET HEADERS
  BASE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/include"
  FILES "${CMAKE_CURRENT_SOURCE_DIR}/include/MinHook.h"
)

target_include_directories(minhook PRIVATE "src/")
target_include_directories(minhook PRIVATE "src/hde/")

set_target_properties(minhook PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
)

if(WIN32)
  set_target_properties(minhook PROPERTIES PREFIX "")
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set_target_properties(minhook PROPERTIES DEBUG_POSTFIX ".x64d")
    set_target_properties(minhook PROPERTIES RELEASE_POSTFIX ".x64")
    set_target_properties(minhook PROPERTIES RELWITHDEBINFO_POSTFIX ".x64")
    set_target_properties(minhook PROPERTIES MINSIZEREL_POSTFIX ".x64")
  else()
    set_target_properties(minhook PROPERTIES DEBUG_POSTFIX ".x32d")
    set_target_properties(minhook PROPERTIES RELEASE_POSTFIX ".x32")
    set_target_properties(minhook PROPERTIES RELWITHDEBINFO_POSTFIX ".x32")
    set_target_properties(minhook PROPERTIES MINSIZEREL_POSTFIX ".x32")
  endif()
else()
  set_target_properties(minhook PROPERTIES PREFIX "lib")
  set_target_properties(minhook PROPERTIES DEBUG_POSTFIX "d")
endif()

################
# CMAKE CONFIG #
################

set(MINHOOK_INSTALL_CMAKEDIR "${CMAKE_INSTALL_DATADIR}/minhook")

configure_package_config_file(
    "cmake/minhook-config.cmake.in"
    "minhook-config.cmake"
  INSTALL_DESTINATION
    "${MINHOOK_INSTALL_CMAKEDIR}"
  PATH_VARS
    CMAKE_INSTALL_INCLUDEDIR
    CMAKE_INSTALL_LIBDIR
)

write_basic_package_version_file(
  "minhook-config-version.cmake"
VERSION
  ${MINHOOK_VERSION}
COMPATIBILITY
  AnyNewerVersion
)

install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/minhook-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/minhook-config-version.cmake"
  DESTINATION
    "${MINHOOK_INSTALL_CMAKEDIR}"
)

###################
#     INSTALL     #
###################

install(TARGETS minhook
        EXPORT minhook-targets
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        FILE_SET HEADERS DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
  EXPORT
    minhook-targets
  NAMESPACE
    minhook::
  DESTINATION
    "${MINHOOK_INSTALL_CMAKEDIR}"
)
