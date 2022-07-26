cmake_minimum_required(VERSION 3.15)

include(CMakePackageConfigHelpers)

set(qr_code_generator_VERSION_MAJOR 1)
set(qr_code_generator_VERSION_MINOR 0)
set(qr_code_generator_VERSION_PATCH 0)
set(qr_code_generator_VERSION ${qr_code_generator_VERSION_MAJOR}.${qr_code_generator_VERSION_MINOR}.${qr_code_generator_VERSION_PATCH})

# Define the library's project
project(
  qr_code_generator
    LANGUAGES C
    VERSION ${qr_code_generator_VERSION}
    DESCRIPTION "QR-Code-Generator CMake wrapper for TEG style libraries (https://github.com/nayuki/QR-Code-generator)"
)

# General compile_commands.json for clangd lsp, has to be above target creation
set(
  CMAKE_EXPORT_COMPILE_COMMANDS ON
  CACHE
    BOOL
    "Generate compile commands database for clang tooling"
    FORCE
)

# Define the library and its alias
add_library(qr_code_generator SHARED)
add_library(qr_code_generator::qr_code_generator ALIAS qr_code_generator)

# Set the library's version
set_target_properties(
  qr_code_generator
    PROPERTIES
      SOVERSION ${qr_code_generator_VERSION_MAJOR}
      VERSION ${qr_code_generator_VERSION}
      DEBUG_POSTFIX "-d"
)

# Enumerate headers
set(
  qr_code_generator_PUBLIC_HEADERS
    qrcodegen.h
)

set_target_properties(
  qr_code_generator
    PROPERTIES 
      PUBLIC_HEADER "${qr_code_generator_PUBLIC_HEADERS}"
)

target_sources(
  qr_code_generator
    PRIVATE
      qrcodegen.c
)

# compiler options
target_compile_options(
  qr_code_generator
    PRIVATE
      $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>:
      -Wall
      -Wextra
      -Wpedantic>
)

# IDEs should put the headers in a nice place
source_group(
  TREE "${TEG_HOME}/include"
  PREFIX "Header Files"
  FILES ${HEADER_LIST}
)

# Configure config package file pased on .in file
configure_package_config_file(
  qr_code_generatorConfig.cmake.in
  ${CMAKE_BINARY_DIR}/cmake/qr_code_generator/qr_code_generatorConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake
)

write_basic_package_version_file(
  ${CMAKE_BINARY_DIR}/cmake/qr_code_generator/qr_code_generatorConfigVersion.cmake
  VERSION ${qr_code_generator_VERSION}
  COMPATIBILITY SameMajorVersion
)

# install library targets
install(
  TARGETS qr_code_generator
  EXPORT qr_code_generatorTargets
  LIBRARY 
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME
    DESTINATION ${CMAKE_INSTALL_BINDIR}
  PUBLIC_HEADER
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/qr_code_generator
)

# install the qr_code_generatorConfig.cmake and qr_code_generatorConfigVersion.cmake
install(
  FILES
    ${CMAKE_BINARY_DIR}/cmake/qr_code_generator/qr_code_generatorConfig.cmake
    ${CMAKE_BINARY_DIR}/cmake/qr_code_generator/qr_code_generatorConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/qr_code_generator
  COMPONENT qr_code_generator_DEVELOPMENT
)

# install the qr_code_generatorTargets.cmake file
install(
  EXPORT qr_code_generatorTargets
  FILE qr_code_generatorTargets.cmake
  NAMESPACE qr_code_generator::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/qr_code_generator
  COMPONENT devel
)

