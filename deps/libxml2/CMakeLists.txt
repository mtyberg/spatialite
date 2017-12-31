cmake_minimum_required(VERSION 3.5)
project( libxml2 )

if (NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-attributes")
endif()


configure_file("${PROJECT_SOURCE_DIR}/config.h.in" ${PROJECT_SOURCE_DIR}/config.h @ONLY)

file( GLOB XML2_HEADERS ${PROJECT_SOURCE_DIR}/*.h)
file( GLOB XML2_SOURCES ${PROJECT_SOURCE_DIR}/*.c )

file( GLOB API_HEADERS ${PROJECT_SOURCE_DIR}/include/libxml/*.h )

add_library( libxml2 STATIC ${XML2_SOURCES} ${XML2_HEADERS})
target_include_directories(libxml2 PUBLIC ${PROJECT_SOURCE_DIR}/include)

target_link_libraries(libxml2 ws2_32)

#set_target_properties(xml2 PROPERTIES PUBLIC_HEADER "${API_HEADERS}")
