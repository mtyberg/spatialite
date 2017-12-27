cmake_minimum_required(VERSION 3.5)
project(spatialite)
set(NAME spatialite)

if (NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    add_compile_options(-Wno-deprecated-declarations -Wno-deprecated-register -Wno-switch)
endif ()


#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Dependencies
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
set(DEPENDENCIES_PATH ${CMAKE_CURRENT_LIST_DIR})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# geos
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(geos geos)
set(GEOS_HEADERS_PATH "geos/include/")
set(GEOS_C_HEADERS_PATH "geos/capi/")
target_include_directories(${NAME} PUBLIC ${GEOS_HEADERS_PATH} ${GEOS_C_HEADERS_PATH})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# proj4
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
set(USE_THREAD OFF CACHE BOOL " ")
set(BUILD_LIBPROJ_SHARED OFF CACHE BOOL " ")
set(PROJ4_TESTS OFF CACHE BOOL " ")
add_subdirectory(proj4 proj4)
set(PROJ_HEADERS_PATH "proj4/src")
target_include_directories(${NAME} PUBLIC ${PROJ_HEADERS_PATH})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# sqlite3
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(sqlite3 sqlite3)


