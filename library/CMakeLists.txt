cmake_minimum_required(VERSION 3.4.1)

project(quickjs-android)
message(NOTICE "CMakelist file for QuickJS Library")
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    option(LEAK_TRIGGER "Add a leak trigger" ON)
else ()
    option(LEAK_TRIGGER "Add a leak trigger" OFF)
endif (CMAKE_BUILD_TYPE STREQUAL "Debug")

message(NOTICE, "bin directory is ${CMAKE_CURRENT_BINARY_DIR}")
add_subdirectory(../quickjs ${CMAKE_CURRENT_BINARY_DIR}/quickjs)
set(CMAKE_VERBOSE_MAKEFILE ON)

set(QUICKJS_ANDROID_SOURCES
        src/main/c/quickjs-jni.c
        src/main/c/java-method.c
        src/main/c/java-object.c
        src/main/c/java-helper.c
)

if (LEAK_TRIGGER)
    set(COMMON_FLAGS -DLEAK_TRIGGER -DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON)
else ()
    set(COMMON_FLAGS -DLEAK_TRIGGER -DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON)
endif (LEAK_TRIGGER)

add_library(quickjs-android SHARED ${QUICKJS_ANDROID_SOURCES})
target_compile_options(quickjs-android PRIVATE ${COMMON_FLAGS})
message(NOTICE, "Compiling quickjs-android ${COMMON_FLAGS}")

#target_link_options(quickjs-android PRIVATE "-Wl,-z,max-page-size=16384 --hash-style=both -llog")
#target_link_options(quickjs-android PRIVATE "-Wl,-z,max-page-size=16384 --hash-style=both")
target_link_options(quickjs-android PRIVATE
        -Wl,-z,max-page-size=16384
        -Wl,--hash-style=both
)
target_link_libraries(quickjs-android PRIVATE quickjs log)
#target_link_libraries(quickjs-android PRIVATE quickjs -Wl,-z,max-page-size=16384 --hash-style=both, -llog )
