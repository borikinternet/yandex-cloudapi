cmake_minimum_required(VERSION "3.0")

project("yandex-cloudapi-protobuf")

set(CMAKE_PREFIX_PATH "~/.local;${CMAKE_SYSTEM_PREFIX_PATH}")

find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${Protobuf_VERSION}")

find_package(absl CONFIG REQUIRED)
message(STATUS "Using absl ${absl_VERSION}")

find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

file(GLOB_RECURSE ALL_PROTO_FILES *.proto)
if (NOT DEFINED LANG OR "${LANG}" STREQUAL "CPP")
    message(STATUS "Generating C++ GRPC sources and headers")
    set(CMAKE_CXX_STANDARD 14)

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
                DEPENDS "${_f}" protobuf::protoc gRPC::grpc_cpp_plugin
        )
        list(APPEND CPP_PROTO_FILES "${_f_name_h}" "${_f_name_cpp}" "${_f_name_grpc_h}" "${_f_name_grpc_cpp}")
    endforeach ()

    add_custom_target(gen_cpp_files
            DEPENDS ${CPP_PROTO_FILES})

    add_library(yandex_cloudapi_obj STATIC ${CPP_PROTO_FILES})
    target_include_directories(yandex_cloudapi_obj
            PRIVATE ${Protobuf_INCLUDE_DIRS}
                    "~/.local/include"
            PUBLIC ${PROJECT_BINARY_DIR})
    target_compile_options(yandex_cloudapi_obj PRIVATE -fPIC)
elseif ("${LANG}" STREQUAL  "GO")
    message(STATUS "Generating Golang GRPC sources and headers")

    set(GO_PROTO_FILES)
    set(PROTO_FILES)
    foreach (_f ${ALL_PROTO_FILES})
        get_filename_component(_path ${_f} DIRECTORY)
        file(RELATIVE_PATH _f_path ${PROJECT_SOURCE_DIR} ${_path})
        get_filename_component(_f_name ${_f} NAME_WE)
        set(_f_name_go "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}.pb.go")
        set(_f_name_grpc_go "${CMAKE_CURRENT_BINARY_DIR}/${_f_path}/${_f_name}_grpc.pb.go")

        add_custom_command(
                OUTPUT "${_f_name_go}" "${_f_name_grpc_go}"
                COMMAND $<TARGET_FILE:protobuf::protoc>
                ARGS
                --go-grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
                --go_out "${CMAKE_CURRENT_BINARY_DIR}"
                -I"${CMAKE_CURRENT_SOURCE_DIR}"
                "${_f}"
                DEPENDS "${_f}" protobuf::protoc
        )

        list(APPEND GO_PROTO_FILES "${_f_name_go}" "${_f_name_grpc_go}")
        list(APPEND PROTO_FILES "${_f}")
    endforeach ()

    add_custom_target(gen_go_files
            DEPENDS ${GO_PROTO_FILES})

endif ()