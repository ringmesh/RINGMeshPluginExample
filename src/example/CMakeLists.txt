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

# Configure project library
add_library(example SHARED "")
set_target_properties(example PROPERTIES OUTPUT_NAME RINGMesh_example FOLDER "Libraries")
target_include_directories(example
    PUBLIC   
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
        $<INSTALL_INTERFACE:include>
)

set(example_include_dir ${PROJECT_SOURCE_DIR}/include/example)
set(example_source_dir ${PROJECT_SOURCE_DIR}/src/example)
target_sources(example
    PRIVATE
        "${example_source_dir}/common.cpp"
        "${example_source_dir}/hello_world.cpp"
    PRIVATE # Could be PUBLIC from CMake 3.3
        "${example_include_dir}/common.h"
        "${example_include_dir}/hello_world.h"
)

target_link_libraries(example 
    PUBLIC 
        RINGMesh::basic
        RINGMesh::geomodel_core
        RINGMesh::io
)

export(TARGETS example 
    NAMESPACE RINGMeshPluginExample:: 
    FILE cmake/RINGMesh_example_target.cmake
)
generate_export_header(example 
    EXPORT_MACRO_NAME example_api 
    EXPORT_FILE_NAME ${PROJECT_BINARY_DIR}/example/export.h
)
install(TARGETS example
    EXPORT example
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
install(EXPORT example
    FILE RINGMesh_example_target.cmake
    NAMESPACE RINGMeshPluginExample::
    DESTINATION cmake
)
