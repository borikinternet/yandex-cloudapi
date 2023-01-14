Using of included CMakeLists.txt example
--
You can import this lib into your project using next CMake instructions:
```cmake
include(FetchContent)
FetchContent_Declare(yandex_grpc
        GIT_REPOSITORY https://github.com/borikinternet/yandex-cloudapi.git)
foreach(entry yandex_grpc)
    FetchContent_GetProperties(${entry})
    set(sd_ ${entry}_SOURCE_DIR)
    set(bd_ ${entry}_BINARY_DIR)
    if (NOT ${entry}_POPULATED)
        FetchContent_Populate(${entry})
        add_subdirectory(${${sd_}} ${${bd_}} EXCLUDE_FROM_ALL)
    endif ()
    set(${entry}_SOURCE_DIR ${${sd_}} PARENT_SCOPE)
    set(${entry}_BINARY_DIR ${${bd_}} PARENT_SCOPE)
endforeach()
```