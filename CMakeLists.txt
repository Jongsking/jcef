# Copyright (c) 2016 The Chromium Embedded Framework Authors. All rights
# reserved. Use of this source code is governed by a BSD-style license that
# can be found in the LICENSE file.

# OVERVIEW
#
# CMake is a cross-platform open-source build system that can generate project
# files in many different formats. It can be downloaded from
# http://www.cmake.org or installed via a platform package manager.
#
# CMake-generated project formats that have been tested with JCEF include:
#
# Linux:      Ninja, Unix Makefiles
# MacOS:      Ninja, Xcode 8+ (x86_64) or Xcode 12.2+ (ARM64)
# Windows:    Ninja, Visual Studio 2015+
#
# Ninja is a cross-platform open-source tool for running fast builds using
# pre-installed platform toolchains (GNU, clang, Xcode or MSVC). It can be
# downloaded from http://martine.github.io/ninja/ or installed via a platform
# package manager.
#
# BUILD REQUIREMENTS
#
# The below requirements must be met to build JCEF.
#
# - CMake version 2.8.12.1 or newer.
#
# - Linux requirements:
#   Currently supported distributions include Debian Wheezy, Ubuntu Precise, and
#   related. Ubuntu 18.04 64-bit is recommended. Newer versions will likely also
#   work but may not have been tested.
#   Required packages include:
#     build-essential
#
# - MacOS requirements:
#   Xcode 8 or newer building on MacOS 10.11 (El Capitan) or newer for x86_64.
#   Xcode 12.2 or newer building on MacOS 10.15.4 (Catalina) or newer for ARM64.
#   Only 64-bit builds are supported. The Xcode command-line tools must also be
#   installed.
#
# - Windows requirements:
#   Visual Studio 2015 Update 2 or newer building on Windows 7 or newer. Visual
#   Studio 2019 and Windows 10 64-bit are recommended.
#
# BUILD EXAMPLES
#
# The below commands will generate project files and create a Debug build of all
# JCEF native targets using CMake and the platform toolchain.
#
# Start by creating and entering the CMake build output directory. The
#`jcef_build` directory name is required by other JCEF tooling (specifically the
# tools/make_distrib.[bat|sh] and tools/run.[bat|sh] scripts) and should not be
# changed.
# > cd path/to/java-cef/src
# > mkdir jcef_build && cd jcef_build
#
# To perform a Linux build using a 32-bit CEF binary distribution on a 32-bit
# Linux platform or a 64-bit CEF binary distribution on a 64-bit Linux platform:
#   Using Unix Makefiles:
#     > cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug ..
#     > make -j4
#
#   Using Ninja:
#     > cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug ..
#     > ninja
#
# To perform a MacOS build using a 64-bit CEF binary distribution:
#   Using the Xcode IDE:
#     > cmake -G "Xcode" -DPROJECT_ARCH="x86_64" ..
#     Open jcef.xcodeproj in Xcode and select Product > Build.
#
#   Using Ninja:
#     > cmake -G "Ninja" -DPROJECT_ARCH="x86_64" -DCMAKE_BUILD_TYPE=Debug ..
#     > ninja
#
# To perform a MacOS build using an ARM64 CEF binary distribution:
#   Using the Xcode IDE:
#     > cmake -G "Xcode" -DPROJECT_ARCH="arm64" ..
#     Open jcef.xcodeproj in Xcode and select Product > Build.
#
#   Using Ninja:
#     > cmake -G "Ninja" -DPROJECT_ARCH="arm64" -DCMAKE_BUILD_TYPE=Debug ..
#     > ninja
#
# To perform a Windows build using a 32-bit CEF binary distribution:
#   Using the Visual Studio 2019 IDE:
#     > cmake -G "Visual Studio 16" -A Win32 ..
#     Open jcef.sln in Visual Studio and select Build > Build Solution.
#
#   Using Ninja with Visual Studio 2019 command-line tools:
#     (this path may be different depending on your Visual Studio installation)
#     > "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars32.bat"
#     > cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug ..
#     > ninja
#
# To perform a Windows build using a 64-bit CEF binary distribution:
#   Using the Visual Studio 2019 IDE:
#     > cmake -G "Visual Studio 16" -A x64 ..
#     Open jcef.sln in Visual Studio and select Build > Build Solution.
#
#   Using Ninja with Visual Studio 2019 command-line tools:
#     (this path may be different depending on your Visual Studio installation)
#     > "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars64.bat"
#     > cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug ..
#     > ninja

#
# Shared configuration.
#

cmake_minimum_required(VERSION 2.8.12.1)

# Only generate Debug and Release configuration types.
set(CMAKE_CONFIGURATION_TYPES Debug Release)

# Project name.
project(jcef)

# Use folders in the resulting project files.
set_property(GLOBAL PROPERTY OS_FOLDERS ON)


#
# CEF configuration.
#

# Specify the CEF distribution version.
if(NOT DEFINED CEF_VERSION)
  set(CEF_VERSION "87.1.13+g481a82a+chromium-87.0.4280.141")
endif()

# Determine the platform.
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin")
  if("${PROJECT_ARCH}" STREQUAL "arm64")
    set(CEF_PLATFORM "macosarm64")
  else()
    set(CEF_PLATFORM "macosx64")
  endif()
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
  if(CMAKE_SIZEOF_VOID_P MATCHES 8)
    set(CEF_PLATFORM "linux64")
  else()
    set(CEF_PLATFORM "linux32")
  endif()
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  if(CMAKE_SIZEOF_VOID_P MATCHES 8)
    set(CEF_PLATFORM "windows64")
  else()
    set(CEF_PLATFORM "windows32")
  endif()
endif()

# Add this project's cmake/ directory to the module path.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Download and extract the CEF binary distribution (executes DownloadCEF.cmake).
include(DownloadCEF)
DownloadCEF("${CEF_PLATFORM}" "${CEF_VERSION}" "${CMAKE_SOURCE_DIR}/third_party/cef")

# Add the CEF binary distribution's cmake/ directory to the module path.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CEF_ROOT}/cmake")

# Load the CEF configuration (executes FindCEF.cmake).
find_package(CEF REQUIRED)


#
# Python configuration.
#

# Support specification of the Python executable path via the command-line.
if(DEFINED ENV{PYTHON_EXECUTABLE})
  file(TO_CMAKE_PATH "$ENV{PYTHON_EXECUTABLE}" PYTHON_EXECUTABLE)
endif()

if(NOT PYTHON_EXECUTABLE)
  unset(PYTHON_EXECUTABLE)

  # Find the python interpreter.
  find_package(PythonInterp)

  if(NOT ${PYTHONINTERP_FOUND})
    message(FATAL_ERROR "A Python installation is required. Set the "
                        "PYTHON_EXECUTABLE environment variable to explicitly "
                        "specify the Python executable path.")
  endif()
endif()

message(STATUS "Using Python: ${PYTHON_EXECUTABLE}")


#
# Java configuration.
#

# Minimum required Java version.
set(JDK_MIN_VERSION 1.7)

set(JAVA_FATAL_ERROR "A Java installation is required. Set the JAVA_HOME "
                     "environment variable to explicitly specify the Java "
                     "installation directory.")

# Find the Java Native Interface (JNI) installation.
find_package(JNI ${JDK_MIN_VERSION})
if(NOT ${JNI_FOUND})
  message(FATAL_ERROR ${JAVA_FATAL_ERROR})
endif()

if(OS_MACOSX)
  # OS X stores the Java binaries separately from the JNI includes/libraries.
  # Find the Java development installation.
  find_package(Java ${JDK_MIN_VERSION} COMPONENTS Development)

  if(NOT ${Java_FOUND})
    message(FATAL_ERROR ${JAVA_FATAL_ERROR})
  endif()

  # Determine the root path for the Java installation.
  # Remove "bin/javac" from the path.
  get_filename_component(JAVA_DIR ${Java_JAVAC_EXECUTABLE} DIRECTORY)
  get_filename_component(JAVA_DIR ${JAVA_DIR} DIRECTORY)
else()
  # Determine the root path for the Java installation.
  # Remove "include" from the path.
  get_filename_component(JAVA_DIR ${JAVA_INCLUDE_PATH} DIRECTORY)
endif()


#
# Post-configuration actions.
#

# Generate the JCEF version header.
message(STATUS "Generating native/jcef_version.h file...")
execute_process(
  COMMAND "${PYTHON_EXECUTABLE}"
          "tools/make_version_header.py"
          "--header"
          "native/jcef_version.h"
          "--cef-path"
          "${CEF_ROOT}"
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  RESULT_VARIABLE EXECUTE_RV
  )
if(NOT EXECUTE_RV STREQUAL "0")
  message(FATAL_ERROR "Execution failed with unexpected result: ${EXECUTE_RV}")
endif()

# Copy the CEF README.txt file to the cmake build directory for use by the
# make_readme.py script.
file(COPY "${CEF_ROOT}/README.txt" DESTINATION "${CMAKE_BINARY_DIR}")

# Download clang-format from Google Storage.
if(OS_WINDOWS)
  set(GS_PLATFORM "win32")
  set(GS_HASHPATH "win/clang-format.exe.sha1")
elseif(OS_MACOSX)
  set(GS_PLATFORM "darwin")
  set(GS_HASHPATH "mac/clang-format.sha1")
elseif(OS_LINUX)
  set(GS_PLATFORM "linux*")
  set(GS_HASHPATH "linux64/clang-format.sha1")
endif()

message(STATUS "Downloading clang-format from Google Storage...")
execute_process(
  COMMAND "${PYTHON_EXECUTABLE}"
          "tools/buildtools/download_from_google_storage.py"
          "--no_resume"
          "--platform=${GS_PLATFORM}"
          "--no_auth"
          "--bucket" "chromium-clang-format"
          "-s" "tools/buildtools/${GS_HASHPATH}"
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  RESULT_VARIABLE EXECUTE_RV
  )
if(NOT EXECUTE_RV STREQUAL "0")
  message(FATAL_ERROR "Execution failed with unexpected result: ${EXECUTE_RV}")
endif()


#
# Include target subdirectories.
#

add_subdirectory(${CEF_LIBCEF_DLL_WRAPPER_PATH} libcef_dll_wrapper)
add_subdirectory(native)


#
# Display configuration settings.
#

PRINT_CEF_CONFIG()

message(STATUS "*** JCEF CONFIGURATION SETTINGS ***")
message(STATUS "Python executable:            ${PYTHON_EXECUTABLE}")
message(STATUS "Java directory:               ${JAVA_DIR}")
message(STATUS "JNI libraries:                ${JNI_LIBRARIES}")
message(STATUS "JNI include directories:      ${JNI_INCLUDE_DIRS}")
