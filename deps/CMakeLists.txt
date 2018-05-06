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
if (("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC"))
	target_include_directories(${NAME} PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/geos/include")
	set(Project_WC_REVISION 999999)
endif()
add_subdirectory(geos geos)
set(GEOS_HEADERS_PATH "geos/include/")
set(GEOS_C_HEADERS_PATH "geos/capi/")
target_include_directories(${NAME} PUBLIC ${GEOS_HEADERS_PATH} ${GEOS_C_HEADERS_PATH})
target_include_directories(${NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/geos/capi)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# proj4
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
set(USE_THREAD OFF CACHE BOOL " ")
set(BUILD_LIBPROJ_SHARED OFF CACHE BOOL " ")
set(PROJ4_TESTS OFF CACHE BOOL " ")
add_subdirectory(proj4 proj4)
remove_definitions(-DMUTEX_stub)
if (("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC"))
	add_definitions( -DMUTEX_win32)
else()
	add_definitions( -DMUTEX_pthread)
endif()
set(PROJ_HEADERS_PATH "proj4/src")
target_include_directories(${NAME} PUBLIC ${PROJ_HEADERS_PATH})

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# sqlite3
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(sqlite3 sqlite3)
set(SQLITE_HEADERS_PATH "sqlite3/src")
target_include_directories(${NAME} PUBLIC ${SQLITE_HEADERS_PATH})

if (("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC"))
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# zlib
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(zlib zlib)
set(ZLIB_HEADERS_PATH "zlib")
target_include_directories(${NAME} PUBLIC ${ZLIB_HEADERS_PATH})
target_include_directories(${NAME} PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/zlib")

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# iconv
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_definitions(-DICONV_CONST=const)
set(BUILD_SHARED_LIBS OFF CACHE BOOL " ")
add_subdirectory(libiconv iconv)
target_include_directories(${NAME} PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/iconv")

#~~~~~~~~~~~~~~~~~~~~~~~~~~~
# xml2
#~~~~~~~~~~~~~~~~~~~~~~~~~~~
add_subdirectory(libxml2 libxml2)
target_include_directories(libxml2 PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/iconv")
target_include_directories(libxml2 PUBLIC ${ZLIB_HEADERS_PATH})
target_include_directories(libxml2 PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/zlib")

endif()

