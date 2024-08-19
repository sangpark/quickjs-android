cmake_minimum_required(VERSION 3.4.1)

project(quickjs C)


if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    option(LEAK_TRIGGER "Add a leak trigger" ON)
else ()
    option(LEAK_TRIGGER "Add a leak trigger" OFF)
endif (CMAKE_BUILD_TYPE STREQUAL "Debug")
#option(LEAK_TRIGGER "Add a leak trigger" FALSE)
message(NOTICE "CMakelist file for QuickJS")

file(STRINGS "quickjs/VERSION" CONFIG_VERSION)
set(CMAKE_VERBOSE_MAKEFILE ON)

include_directories(quickjs)

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CONFIG_CC clang)
else()
    set(CONFIG_CC gcc)
endif()

set(COMMON_FLAGS -D_GNU_SOURCE -DCONFIG_VERSION=\"${CONFIG_VERSION}\" -DCONFIG_CC=\"${CONFIG_CC}\" -DCONFIG_PREFIX=\"/usr/local\" -DCONFIG_BIGNUM -DANDROID_PLATFORM=android-18 -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON -DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON )
message(NOTICE "Setting the common flags for the quickjs compilation")
set(QUICKJS_LIB_SOURCES
        quickjs/quickjs.c
        quickjs/libregexp.c
        quickjs/libunicode.c
        quickjs/libbf.c
        quickjs/cutils.c
        )

set(QJS_LIB_SOURCES
        quickjs/quickjs-libc.c
        ${QUICKJS_LIB_SOURCES}
        )

set(QJS_SOURCES
        quickjs/qjs.c
        repl.c
        qjscalc.c
        ${QJS_LIB_SOURCES}
        )

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(COMMON_FLAGS ${COMMON_FLAGS} -DDUMP_LEAKS)
endif (CMAKE_BUILD_TYPE STREQUAL "Debug")

message(NOTICE "LEAK TRIGGER ${LEAK_TRIGGER}")
if (LEAK_TRIGGER)
    # Use printf as leak_trigger
    set(COMMON_FLAGS ${COMMON_FLAGS})
#    set(COMMON_FLAGS ${COMMON_FLAGS} -Dprintf=leak_trigger)
endif (LEAK_TRIGGER)
message(NOTICE "Setting the CMAKE SHARAED LINKER FLAGS ${COMMON_FLAGS}")
# Camouflage executable files as shared libraries to make apk include them
#set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--hash-style=both")

add_executable(qjs ${QJS_SOURCES})
target_compile_options(qjs PRIVATE ${COMMON_FLAGS})
set_target_properties(qjs PROPERTIES OUTPUT_NAME libqjs.so)

add_executable(qjsc quickjs/qjsc.c ${QJS_LIB_SOURCES})
target_compile_options(qjsc PRIVATE ${COMMON_FLAGS})
set_target_properties(qjsc PROPERTIES OUTPUT_NAME libqjsc.so)

add_executable(run-test262 quickjs/run-test262.c ${QJS_LIB_SOURCES})
target_compile_options(run-test262 PRIVATE ${COMMON_FLAGS})
set_target_properties(run-test262 PROPERTIES OUTPUT_NAME librun-test262.so)

add_library(quickjs SHARED ${QUICKJS_LIB_SOURCES})
add_library(quickjs_static STATIC ${QUICKJS_LIB_SOURCES}) # Create the static library .a file
target_compile_options(quickjs PRIVATE ${COMMON_FLAGS})
target_include_directories(quickjs PUBLIC quickjs .)
find_library(log-lib log)

# Link the log library to the quickjs library
target_link_libraries(quickjs PRIVATE ${log-lib})

message(NOTICE "Setting Linkages")
#target_link_options(quickjs PRIVATE
#        -Wl,-z,max-page-size=16384
#        -Wl,--hash-style=both
#)
#target_link_libraries(quickjs PRIVATE log)

#target_link_options(quickjs PRIVATE -Wl,-z,max-page-size=16384 --hash-style=both)
#target_link_libraries(quickjs -llog)

#target_link_libraries(quickjs PRIVATE -Wl,--max-page-size=16384,--hash-style=both)