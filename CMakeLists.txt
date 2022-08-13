cmake_minimum_required(VERSION "3.0")

project("yandex-cloudapi-protobuf")

find_package(Protobuf REQUIRED)
file(GLOB_RECURSE ALL_PROTO_FILES *.proto)
set(CPP_PROTO_FILES)
foreach (_f ${ALL_PROTO_FILES})
    get_filename_component(_path ${_f} DIRECTORY)
    file(RELATIVE_PATH _f_path ${PROJECT_SOURCE_DIR} ${_path})
    protobuf_generate(
            LANGUAGE cpp
            PROTOS ${_f}
            OUT_VAR _f_results
    )
    list(APPEND CPP_PROTO_FILES ${_f_results})
endforeach ()

add_library(yandex_cloudapi_obj STATIC ${CPP_PROTO_FILES})
target_include_directories(yandex_cloudapi_obj
        PRIVATE ${Protobuf_INCLUDE_DIRS}
        PUBLIC ${PROJECT_BINARY_DIR})