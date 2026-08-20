
cmake_minimum_required(VERSION 3.16)

project(bs7535UiAarch64V01c01Glibc LANGUAGES C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

find_package(Threads REQUIRED)

include(cmake/lvgl_prune_unused.cmake)

set(LV_BUILD_DEMOS OFF CACHE BOOL "Build LVGL demos")
set(LV_BUILD_EXAMPLES OFF CACHE BOOL "Build LVGL examples")

add_subdirectory(freetype)
lvgl_sync_build_options_from_conf("${PROJECT_SOURCE_DIR}/lv_conf.h")
add_subdirectory(lvgl)
add_library(bs7535::lvgl ALIAS lvgl)
lvgl_prune_unused_sources(lvgl "${PROJECT_SOURCE_DIR}/lv_conf.h")

target_compile_options(lvgl
        PRIVATE
        $<$<COMPILE_LANG_AND_ID:C,GNU>:-Wno-psabi>
)

target_include_directories(lvgl
        PUBLIC
        ${PROJECT_SOURCE_DIR}
)

target_link_libraries(lvgl
        PUBLIC
        bs7535::freetype
)

target_compile_definitions(lvgl
        PUBLIC
        LV_CONF_INCLUDE_SIMPLE=1
        PRIVATE
        FT2_BUILD_LIBRARY
)

add_subdirectory(ui)

add_executable(${PROJECT_NAME})

target_sources(${PROJECT_NAME}
        PRIVATE
        main.c
)

target_link_libraries(${PROJECT_NAME}
        PRIVATE
        bs7535::ui_app
        Threads::Threads
)

target_compile_options(${PROJECT_NAME}
        PRIVATE
        $<$<COMPILE_LANG_AND_ID:C,GNU,Clang>:-Wall>
        $<$<COMPILE_LANG_AND_ID:C,GNU,Clang>:-Wextra>
        $<$<COMPILE_LANG_AND_ID:C,GNU,Clang>:-Wno-unused>
)

set_target_properties(${PROJECT_NAME} PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin"
)

if(NOT CMAKE_CROSSCOMPILING)
    add_custom_target(run
            COMMAND $<TARGET_FILE:${PROJECT_NAME}>
            DEPENDS ${PROJECT_NAME}
            USES_TERMINAL
    )
endif()
