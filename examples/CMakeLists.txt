link_libraries(spatialite)

include_directories(${spatialite_INCLUDE_DIRS} "${PROJECT_SOURCE_DIR}/srs/headers")

if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()


add_executable(demo1 WIN32 MACOSX_BUNDLE demo1.c )
add_executable(demo2 WIN32 MACOSX_BUNDLE demo2.c )
add_executable(demo3 WIN32 MACOSX_BUNDLE demo3.c )
add_executable(demo4 WIN32 MACOSX_BUNDLE demo4.c )
add_executable(demo5 WIN32 MACOSX_BUNDLE demo5.c )

set(WINDOWS_BINARIES boing gears heightmap particles simple splitview wave)

if (APPLE)
    set_target_properties(demo1 PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "demo1")
    set_target_properties(demo2 PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "demo2")
    set_target_properties(demo3 PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "demo3")
    set_target_properties(demo4 PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "demo4")
    set_target_properties(demo5 PROPERTIES MACOSX_BUNDLE_BUNDLE_NAME "demo5")
endif()
