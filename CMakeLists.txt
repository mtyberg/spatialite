cmake_minimum_required(VERSION 3.5)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# CCache preparation
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# This should happen before project()
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

project(spatialite)
set(NAME spatialite)

if (NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    add_compile_options(-Wno-error=uninitialized -Wno-error=implicit-function-declaration -Wno-error=int-to-void-pointer-cast -Wno-error=unused-parameter -Wno-error=unused-variable -Wno-error=self-assign -Wno-error=unused-parameter -Wno-error=sometimes-uninitialized -Wno-error=logical-not-parentheses -Wno-error=unused-function)
else()
	add_definitions(-DYY_NO_UNISTD_H)
endif ()

option(ENABLE_GEOPACKAGE "Enable GeoPackage" ON)
option(ENABLE_LIBXML2 "Enable libxml2" ON)
option(HAVE_LIBSQLITE3 "Enable libsqlite3" ON)
option(OMIT_FREEXL "Omit Freexl" ON)
option(GEOS_ADVANCED "Enable Geo Advanced" ON)
option(HAVE_ICONV_H "Enable iconv" ON)
option(HAVE_LIBZ "Enable libz" ON)
option(VERSION "Enable VERSION" ON)
set(VERSIONSTRING "4.3.0a")
option(TARGET_CPU "TARGET_CPU" ON)
set(TARGET_CPU_STRING "arch")

include (CheckIncludeFiles)
include (CheckLibraryExists)
include (CheckFunctionExists)

check_include_files (dlfcn.h HAVE_DLFCN_H)
check_include_files (fcntl.h HAVE_FCNTL_H)
check_include_files (inttypes.h HAVE_INTTYPES_H)
check_include_files (float.h HAVE_FLOAT_H)
check_include_files (math.h HAVE_MATH_H)
check_include_files (memory.h HAVE_MEMORY_H)
check_include_files (stdint.h HAVE_STDINT_H)
check_include_files (stdio.h HAVE_STDIO_H)
check_include_files (stdlib.h HAVE_STDLIB_H)
check_include_files (string.h HAVE_STRING_H)
check_include_files (strings.h HAVE_STRINGS_H)
check_include_files (sys/stat.h HAVE_SYS_STAT_H)
check_include_files (sys/types.h HAVE_SYS_TYPES_H)
check_include_files (sys/time.h HAVE_SYS_TIME_H)
check_include_files("sys/types.h;time.h" TIME_WITH_SYS_TIME)
check_include_files (unistd.h HAVE_UNISTD_H)
check_include_files("stdlib.h;stdarg.h;string.h;float.h" STDC_HEADERS)
check_include_files (zlib.h HAVE_ZLIB_H)
check_include_files (sqlite3ext.h HAVE_SQLITE3EXT_H)
check_include_files (sqlite3.h HAVE_SQLITE3_H)

check_function_exists(fdatasync HAVE_FDATASYNC)
check_function_exists(ftruncate HAVE_FTRUNCATE)
check_function_exists(getcwd HAVE_GETCWD)
check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
check_function_exists(localtime_r HAVE_LOCALTIME_R)
check_function_exists(memmove HAVE_MEMMOVE)
check_function_exists(memset HAVE_MEMSET)
check_function_exists(sqrt HAVE_SQRT)
check_function_exists(strcasecmp HAVE_STRCASECMP)
check_function_exists(strerror HAVE_STRERROR)
check_function_exists(strftime HAVE_STRFTIME)
check_function_exists(strncasecmp HAVE_STRNCASECMP)
check_function_exists(strstr HAVE_STRSTR)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Parameters
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
set(CMAKE_CXX_STANDARD 11)

# Project paths
set(HEADERS_PATH "${PROJECT_SOURCE_DIR}/src/headers")
set(SOURCES_PATH "${PROJECT_SOURCE_DIR}/src/spatialite")
set(DEPENDENCIES_RELATIVE_PATH "deps")

if (NOT IOS AND NOT ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC"))
    option(ENABLE_TF_SHARED "Build shared libraries" ON)
    if (ENABLE_TF_SHARED)
        message("BUILD_SHARED_LIBS ON")
        set(BUILD_SHARED_LIBS ON)
    else ()
        message("BUILD_SHARED_LIBS OFF")
        set(BUILD_SHARED_LIBS OFF)
    endif ()
endif()
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Common settings
#~~~~~~~~~~~~~~~~~~~~~~~~~~~

set(LIB_SUFFIX_TXT "Define suffix of directory name (32/64)")
set(EXEC_INSTALL_PREFIX_TXT "Base directory for executables and libraries")
set(BIN_INSTALL_DIR_TXT "The subdirectory to the binaries prefix (default prefix/bin)")
set(LIB_INSTALL_TXT "The subdirectory relative to the install prefix where libraries will be installed (default is /lib${LIB_SUFFIX}")
set(INCLUDE_INSTALL_DIR_TXT "The subdirectory to the header prefix")

set(LIB_SUFFIX "" CACHE STRING "${LIB_SUFFIX_TXT}")
set(EXEC_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}" CACHE PATH "${EXEC_INSTALL_PREFIX_TXT}" FORCE)
set(BIN_INSTALL_DIR "${EXEC_INSTALL_PREFIX}/bin" CACHE PATH "${BIN_INSTALL_DIR_TXT}" FORCE)
set(LIB_INSTALL_DIR "${EXEC_INSTALL_PREFIX}/lib${LIB_SUFFIX}" CACHE PATH "${LIB_INSTALL_TXT}" FORCE)
set(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "${INCLUDE_INSTALL_DIR_TXT}" FORCE)

SET(CMAKE_POLICY_DEFAULT_CMP0008 NEW) # fullpath libs linking 
SET(CMAKE_POLICY_DEFAULT_CMP0042 NEW) # RPATH

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Macros
#~~~~~~~~~~~~~~~~~~~~~~~~~~~

# Enable all warnings for the given target and treat them as error.
macro(lhs_target_configure_warnings ARG_TARGET_NAME)
  if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
    # using Clang
  elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # using GCC
  elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
    # using Intel C++
  elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    # using Visual Studio C++
    set_property(TARGET ${ARG_TARGET_NAME} APPEND_STRING PROPERTY COMPILE_FLAGS /W3 )
  endif()
endmacro(lhs_target_configure_warnings)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Targets
#~~~~~~~~~~~~~~~~~~~~~~~~~~~

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Main library

# Sources and headers
file(GLOB_RECURSE headers "${HEADERS_PATH}/*.h")
file(GLOB_RECURSE sources "${SOURCES_PATH}/*.c")
file(GLOB_RECURSE sources_versioninfo "${PROJECT_SOURCE_DIR}/src/versioninfo/*.c")
file(GLOB_RECURSE sources_connectonCache "${PROJECT_SOURCE_DIR}/src/connection_cache/*.c")
file(GLOB_RECURSE sources_dxf "${PROJECT_SOURCE_DIR}/src/dxf/*.c")
file(GLOB_RECURSE sources_shapefiles "${PROJECT_SOURCE_DIR}/src/shapefiles/*.c")
file(GLOB_RECURSE sources_spatialite "${PROJECT_SOURCE_DIR}/src/spatialite/*.c")
file(GLOB_RECURSE sources_gaiaaux "${PROJECT_SOURCE_DIR}/src/gaiaaux/*.c")
file(GLOB_RECURSE sources_gaiaexif "${PROJECT_SOURCE_DIR}/src/gaiaexif/*.c")
file(GLOB_RECURSE sources_gaiageo "${PROJECT_SOURCE_DIR}/src/gaiageo/gg_*.c")
file(GLOB_RECURSE sources_geopackage "${PROJECT_SOURCE_DIR}/src/geopackage/*.c")
file(GLOB_RECURSE sources_md5 "${PROJECT_SOURCE_DIR}/src/md5/*.c")
file(GLOB sources_srsinit "${PROJECT_SOURCE_DIR}/src/srsinit/*.c")
file(GLOB_RECURSE sources_virtualtext "${PROJECT_SOURCE_DIR}/src/virtualtext/*.c")
file(GLOB_RECURSE sources_wfs "${PROJECT_SOURCE_DIR}/src/wfs/*.c")

# Library parameters
set(TF_FRAMEWORK_VERSION "A")
set(TF_LIB_MAJOR_VERSION "1")
set(TF_LIB_MINOR_VERSION "0")
set(TF_LIB_PATCH_VERSION "0")
set(TF_LIB_VERSION_STRING "${TF_LIB_MAJOR_VERSION}.${TF_LIB_MINOR_VERSION}.${TF_LIB_PATCH_VERSION}")

set(TF_SOVERSION_CURRENT 1)
set(TF_SOVERSION_REVISION 0)
set(TF_SOVERSION_AGE 0)

math(EXPR TF_SOVERSION_MAJOR "${TF_SOVERSION_CURRENT} - ${TF_SOVERSION_AGE}")
math(EXPR TF_SOVERSION_MINOR "${TF_SOVERSION_AGE}")
math(EXPR TF_SOVERSION_PATCH "${TF_SOVERSION_REVISION}")

set(TF_SOVERSION ${TF_SOVERSION_MAJOR}.${TF_SOVERSION_MINOR}.${TF_SOVERSION_PATCH})

message("LIB_INSTALL_DIR: ${LIB_INSTALL_DIR}")

message(STATUS "Generating Spatialite ${PROJECT_SOURCE_DIR}/config.h")
configure_file(${PROJECT_SOURCE_DIR}/config.h.cmake ${PROJECT_SOURCE_DIR}/config.h)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	configure_file(${PROJECT_SOURCE_DIR}/config-msvc.h ${PROJECT_SOURCE_DIR}/config.h @ONLY)
endif()

# Define the lib
add_library(${NAME}
        STATIC
        ${headers}
        ${sources}
        ${sources_versioninfo}
        ${sources_connectonCache}
        ${sources_dxf}
        ${sources_shapefiles}
        ${sources_spatialite}
        ${sources_gaiaaux}
        ${sources_gaiaexif}
        ${sources_gaiageo}
        ${sources_geopackage}
        ${sources_md5}
        ${sources_srsinit}
        ${sources_virtualtext}
        ${sources_wfs}
        ${resources}
        )

# Proper EXPORT setting for the library
target_compile_definitions(${NAME} PRIVATE -DUSE_IMPORT_EXPORT)

# enable all warnings
lhs_target_configure_warnings(${NAME})

# Make full path includes work for project headers
include_directories(${HEADERS_PATH})
include_directories(${PROJECT_SOURCE_DIR})

target_include_directories(${NAME} PUBLIC ${PROJECT_SOURCE_DIR})

# Link dependencies
target_link_libraries (${NAME} geos-static)
target_link_libraries (${NAME} geos_c)
target_link_libraries (${NAME} proj)
target_link_libraries (${NAME} sqlite3)

if (("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC"))
    target_link_libraries (${NAME} zlibstatic)
    target_link_libraries (${NAME} libiconv)
    target_link_libraries(${NAME} libxml2)
else()
    find_package(LibXml2 REQUIRED)
    include_directories(${LIBXML2_INCLUDE_DIR})
    target_link_libraries(${NAME} ${LIBXML2_LIBRARIES})

    find_package(ZLIB REQUIRED)
    target_link_libraries(${NAME} ${ZLIB_LIBRARIES})

    find_library(Iconv_LIBRARY Iconv)
    target_link_libraries(${NAME} ${Iconv_LIBRARY})
endif()


target_compile_features(${NAME} PUBLIC cxx_strong_enums)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Dependencies
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(${DEPENDENCIES_RELATIVE_PATH})

add_subdirectory(examples)
