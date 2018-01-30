# Copyright (c) 2012-2018, Association Scientifique pour la Geologie et ses
# Applications (ASGA). All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
#     * Neither the name of ASGA nor the
#       names of its contributors may be used to endorse or promote products
#       derived from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL ASGA BE LIABLE FOR ANY DIRECT,
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
# THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#     http://www.ring-team.org
#
#     RING Project
#     Ecole Nationale Superieure de Geologie - GeoRessources
#     2 Rue du Doyen Marcel Roubault - TSA 70605
#     54518 VANDOEUVRE-LES-NANCY
#     FRANCE

#------------------------------------------------------------------------------------------------
# Root CMakeList of RINGMeshPluginExample project
#------------------------------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.1)

# Configuration modes - Debug and Release by default + RelWithDebInfo.
# RelWithDebInfo is mandatory to debug some plugins in debug mode to
# avoid issues in particular in the STL.
set(CMAKE_CONFIGURATION_TYPES Debug Release RelWithDebInfo CACHE TYPE INTERNAL FORCE)

# RINGMesh install directory path
set(RINGMesh_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/../RINGMesh/build/Release/ringmesh" 
    CACHE PATH "Path to RINGMesh install directory (mandatory dependency)")

# Download single data file
set(data_file ${CMAKE_SOURCE_DIR}/data/modelA1.ml)
if(NOT EXISTS ${data_file})
    file(DOWNLOAD
        https://raw.githubusercontent.com/ringmesh/RINGMeshData/master/modelA1.ml
        ${data_file}
        SHOW_PROGRESS
    )
endif()

#---------------------------------------------------------------------------------
# Optional section used to simplify multiple project configuration
# Generate configuration directories for single-configuration generators (Make)
# and run cmake configuration command in each one of them
if(CMAKE_GENERATOR STREQUAL "Unix Makefiles")
    # If the CMAKE_BUILD_TYPE is no defined
    # i.e. at first run of that file
    if(NOT CMAKE_BUILD_TYPE)
      # For all configuration types
      foreach(config IN LISTS CMAKE_CONFIGURATION_TYPES)
          project(MULTI_CONFIG NONE)
          # The binary directory for this configuration
          set(project_binary_dir_config  ${PROJECT_BINARY_DIR}/${config})

          # Create the directory
          file(MAKE_DIRECTORY ${project_binary_dir_config})

          # Launch cmake for this configuration
          # by specifying the CMAKE_BUILD_TYPE and the PROJECT _BINARY_DIR,
          # and pass on all the options
          execute_process(
             COMMAND ${CMAKE_COMMAND} ${CMAKE_SOURCE_DIR}
                -G ${CMAKE_GENERATOR}
                -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
                -DCMAKE_BUILD_TYPE=${config}
                -DRINGMesh_INSTALL_PREFIX:PATH=${RINGMesh_INSTALL_PREFIX}
                WORKING_DIRECTORY ${project_binary_dir_config})
      endforeach()

      # Get out when all configurations have been run
      # We do not want to execute the rest of the file in that case
      return()
    endif()
endif(CMAKE_GENERATOR STREQUAL "Unix Makefiles")

#------------------------------------------------------------------------------------------------
# Define the project RINGMeshPluginExample
project(RINGMeshPluginExample)

# Turn on the ability to create folders to organize projects and files
# It creates "CMakePredefinedTargets" folder by default and adds CMake
# defined projects like INSTALL.vcproj and ZERO_CHECK.vcproj
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Define version number
# It is then exported to the configuration file
set(RINGMeshPluginExample_VERSION_MAJOR 1)
set(RINGMeshPluginExample_VERSION_MINOR 0)
set(RINGMeshPluginExample_VERSION_PATCH 0)
set(RINGMeshPluginExample_VERSION 
    ${RINGMeshPluginExample_VERSION_MAJOR}.${RINGMeshPluginExample_VERSION_MINOR}.${RINGMeshPluginExample_VERSION_PATCH}
)

# Ensure C++11 standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build configuration
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)

# Install configuration
set(CMAKE_MACOSX_RPATH ON)
set(CMAKE_INSTALL_RPATH ".")
# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# Exports RINGMeshPluginExample target
include(CMakePackageConfigHelpers)
include(GenerateExportHeader)
configure_package_config_file(
    cmake/RINGMeshPluginExampleConfig.cmake.in 
    ${CMAKE_BINARY_DIR}/lib/cmake/RINGMeshPluginExample/RINGMeshPluginExampleConfig.cmake
    INSTALL_DESTINATION lib/cmake/RINGMeshPluginExample
    PATH_VARS RINGMesh_INSTALL_PREFIX
)
install(
    FILES ${CMAKE_BINARY_DIR}/lib/cmake/RINGMeshPluginExample/RINGMeshPluginExampleConfig.cmake 
    DESTINATION lib/cmake/RINGMeshPluginExample
)
install(DIRECTORY include/example DESTINATION include)

# Import RINGMesh targets
find_package(RINGMesh REQUIRED PATHS ${RINGMesh_INSTALL_PREFIX}/lib/cmake/RINGMesh)

# Configure project library
add_subdirectory(src/example)

# Configure testing framework
enable_testing()
add_subdirectory(tests)

#------------------------------------------------------------------------------------------------
# Configure CPack

set(CPACK_PACKAGE_NAME ${CMAKE_PROJECT_NAME})
set(CPACK_PACKAGE_VERSION_MAJOR ${RINGMeshPluginExample_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${RINGMeshPluginExample_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${RINGMeshPluginExample_VERSION_PATCH})
set(CPACK_PACKAGE_VENDOR "RING-TEAM (www.ring-team.org)")

set(CPACK_SOURCE_GENERATOR "ZIP")
set(CPACK_SOURCE_IGNORE_FILES "/build/;/.git/;/_CPack_Packages/")

if(WIN32)
    set(CPACK_GENERATOR "ZIP")
else()
    set(CPACK_GENERATOR "TGZ")
endif()


# This must always be last!
include(CPack)
