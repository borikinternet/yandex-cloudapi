cmake_minimum_required(VERSION "3.0")

project("yandex-cloudapi-protobuf")

find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${Protobuf_VERSION}")

find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

file(GLOB_RECURSE ALL_PROTO_FILES *.proto)
set(CPP_PROTO_FILES)
foreach (_f ${ALL_PROTO_FILES})
    get_filename_component(_path ${_f} DIRECTORY)
    file(RELATIVE_PATH _f_path ${PROJECT_SOURCE_DIR} ${_path})
    get_filename_component(_f_name ${_f} NAME_WE)
    set(_f_name_h "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}.pb.h")
    set(_f_name_cpp "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}.pb.cc")
    set(_f_name_grpc_h "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}.grpc.pb.h")
    set(_f_name_grpc_cpp "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}.grpc.pb.cc")

    add_custom_command(
            OUTPUT "${_f_name_h}" "${_f_name_cpp}" "${_f_name_grpc_h}" "${_f_name_grpc_cpp}"
            COMMAND $<TARGET_FILE:protobuf::protoc>
            ARGS
            --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
            --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
            -I "${CMAKE_CURRENT_SOURCE_DIR}"
            --plugin=protoc-gen-grpc=$<TARGET_FILE:gRPC::grpc_cpp_plugin>
            "${_f}"
            DEPENDS "${_f}"
    )
    list(APPEND CPP_PROTO_FILES "${_f_name_h}" "${_f_name_cpp}" "${_f_name_grpc_h}" "${_f_name_grpc_cpp}")
endforeach ()

add_library(yandex_cloudapi_obj STATIC ${CPP_PROTO_FILES})
target_include_directories(yandex_cloudapi_obj
        PRIVATE ${Protobuf_INCLUDE_DIRS}
        PUBLIC ${PROJECT_BINARY_DIR})