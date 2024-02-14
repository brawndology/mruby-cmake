cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

project(
  mruby
  VERSION 3.2.0
  LANGUAGES C
)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR
    "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# ---- Dependencies ----

include(cmake/CPM.cmake)

CPMAddPackage("gh:TheLartians/PackageProject.cmake@1.8.0")

# ---- Library ----

add_compile_definitions(MRB_NO_PRESYM)

file(GLOB_RECURSE headers CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/include/mruby/*.h"
)
file(GLOB_RECURSE sources CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/source/*.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/mrbgems/mruby-bigint/core/*.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/mrbgems/mruby-compiler/core/*.c"
)

add_library(${PROJECT_NAME} STATIC ${headers} ${sources})
set_target_properties(${PROJECT_NAME} PROPERTIES C_STANDARD 23)

# being a cross-platform target, we enforce standards conformance on MSVC
target_compile_options(${PROJECT_NAME} PUBLIC "$<$<COMPILE_LANG_AND_ID:C,MSVC>:/permissive->")

# Link dependencies
#target_link_libraries(${PROJECT_NAME} PRIVATE )

target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                         $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)

# ---- Create an installable target ----
# this allows users to install and find the library via `find_package()`.

# the location where the project's version header will be placed should match the project's regular
# header paths
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}-${PROJECT_VERSION}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  COMPATIBILITY SameMajorVersion
  DEPENDENCIES ""
)
