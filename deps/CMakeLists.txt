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
# Catch Unit Tests lib
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
#add_library(Catch INTERFACE)
#set(CATCH_INCLUDE_DIR "${DEPENDENCIES_PATH}/catch/single_include")
#target_include_directories(Catch INTERFACE ${CATCH_INCLUDE_DIR})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# geos
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(geos geos)
set(GEOS_HEADERS_PATH "geos/include/")
target_include_directories(${NAME} PUBLIC ${GEOS_HEADERS_PATH})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# proj4
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
set(USE_THREAD OFF CACHE BOOL " ")
add_subdirectory(proj4 proj4)
set(PROJ_HEADERS_PATH "proj4/src")
target_include_directories(${NAME} PUBLIC ${PROJ_HEADERS_PATH})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# sqlite3
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(sqlite3 sqlite3)


