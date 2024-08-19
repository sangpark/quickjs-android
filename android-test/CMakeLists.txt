cmake_minimum_required(VERSION 3.4.1)

project(quickjs-android-test)

add_subdirectory(../quickjs ${CMAKE_CURRENT_BINARY_DIR}/quickjs)
message(NOTICE "CMakelist file for Android Test")

target_link_options(quickjs PRIVATE
        -Wl,-z,max-page-size=16384
        -Wl,--hash-style=both
)

