cmake_minimum_required(VERSION "3.0")

project("yandex-cloudapi-protobuf")

find_package(Protobuf REQUIRED)
file(GLOB_RECURSE ALL_PROTO_FILES yandex/*.proto)
set(PROTO_FILES)
foreach (_f ${ALL_PROTO_FILES})
    get_filename_component(_path ${_f} DIRECTORY)
    file(RELATIVE_PATH _f_path ${PROJECT_SOURCE_DIR} ${_path})
    protobuf_generate(
            LANGUAGE cpp
            PROTOS ${_f}
            OUT_VAR _f_results
            IMPORT_DIRS third_party/googleapis
    )
    list(APPEND PROTO_FILES ${_f_results})
endforeach ()

add_custom_target(yandex_cloudapi_cpp ALL
        DEPENDS ${PROTO_FILES})