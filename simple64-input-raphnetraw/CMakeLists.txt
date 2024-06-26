cmake_minimum_required(VERSION 3.26)
set(CMAKE_OSX_DEPLOYMENT_TARGET "11.0")

project(simple64-input-raphnetraw LANGUAGES C)

if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
    message("Interprocedural optimizations enabled")
endif()

include_directories(../mupen64plus-core/src/api)

find_package(PkgConfig REQUIRED)

add_library(simple64-input-raphnetraw SHARED
    src/plugin_front.c
    src/plugin_back.c
    src/gcn64lib.c
    src/gcn64.c
    src/hexdump.c
)

if (WIN32)
    target_sources(simple64-input-raphnetraw PRIVATE
        src/osal_dynamiclib_win32.c
    )
    pkg_check_modules(HIDAPI REQUIRED hidapi)
elseif (APPLE)
    target_sources(simple64-input-raphnetraw PRIVATE
        src/osal_dynamiclib_unix.c
    )
    pkg_check_modules(HIDAPI REQUIRED hidapi)
elseif (UNIX)
    target_sources(simple64-input-raphnetraw PRIVATE
        src/osal_dynamiclib_unix.c
    )
    pkg_check_modules(HIDAPI REQUIRED hidapi-hidraw)
endif ()

set_property(TARGET simple64-input-raphnetraw PROPERTY C_VISIBILITY_PRESET hidden)
set_property(TARGET simple64-input-raphnetraw PROPERTY CXX_VISIBILITY_PRESET hidden)
set_property(TARGET simple64-input-raphnetraw PROPERTY VISIBILITY_INLINES_HIDDEN ON)
set_target_properties(simple64-input-raphnetraw PROPERTIES PREFIX "")
target_compile_options(simple64-input-raphnetraw PRIVATE ${HIDAPI_CFLAGS} -Ofast)
target_link_options(simple64-input-raphnetraw PRIVATE -Ofast)
if (CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    target_compile_options(simple64-input-raphnetraw PRIVATE -march=armv8-a)
    target_link_options(simple64-input-raphnetraw PRIVATE -march=armv8-a)
else()
    target_compile_options(simple64-input-raphnetraw PRIVATE -march=x86-64-v3)
    target_link_options(simple64-input-raphnetraw PRIVATE -march=x86-64-v3)
endif()

if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    target_link_options(simple64-input-raphnetraw PRIVATE -fuse-ld=lld)
endif()

if (NOT APPLE AND NOT WIN32)
    target_link_options(simple64-input-raphnetraw PRIVATE -Wl,-Bsymbolic -Wl,-Bsymbolic-functions)
endif ()

target_link_directories(simple64-input-raphnetraw PRIVATE ${HIDAPI_LIBRARY_DIRS})
target_link_libraries(simple64-input-raphnetraw PRIVATE ${HIDAPI_LIBRARIES})
