cmake_minimum_required(VERSION 3.18.1)

project(launcherfix)

add_library(launcherfix SHARED launcher-fix.cpp)

target_compile_features(launcherfix PUBLIC cxx_std_20)
set_target_properties(launcherfix PROPERTIES CXX_EXTENSIONS OFF)

include(FetchContent)

option(USE_TULIPHOOK "enables choice of hooking library" OFF)
if (USE_TULIPHOOK)
    FetchContent_Declare(TulipHook
        GIT_REPOSITORY https://github.com/geode-sdk/TulipHook.git
        GIT_TAG 0021301fcea265416fa8eb5372694931c1d410cb
    )

    set(TULIP_LINK_SOURCE ON CACHE INTERNAL "")
    FetchContent_MakeAvailable(TulipHook)

    target_compile_definitions(launcherfix PRIVATE -DUSE_TULIPHOOK=1)

    target_link_libraries(launcherfix PRIVATE TulipHook TulipHookInclude)
else()
    FetchContent_Declare(dobby
        GIT_REPOSITORY https://github.com/jmpews/Dobby
        GIT_TAG 6a80a61ea062424d1595613058a174c64454fa57
    )

    FetchContent_GetProperties(dobby)
    if(NOT dobby_POPULATED)
        FetchContent_Populate(dobby)

        # fixes Dobby build
        # add_compile_definitions(typeof=__typeof__)
        set(DOBBY_DEBUG OFF CACHE INTERNAL "" FORCE)
        set(DOBBY_GENERATE_SHARED OFF CACHE INTERNAL "" FORCE)

        add_subdirectory("${dobby_SOURCE_DIR}" ${dobby_BINARY_DIR} EXCLUDE_FROM_ALL)
        target_include_directories(launcherfix PRIVATE ${dobby_SOURCE_DIR}/include)
        target_link_libraries(launcherfix PRIVATE dobby)
    endif()
endif()

target_link_libraries(launcherfix PRIVATE log)
