if( NOT ANDROID )
set(BUILD_PROTOBUF_C_COMPILER ON
    CACHE BOOL "build protobuf-c compiler (aka protoc-c)")
endif()

if ( BUILD_PROTOBUF_C_COMPILER )
    find_package(Protobuf REQUIRED)
    set(PROTOC_C_SOURCES
        google/protobuf/compiler/c/c_service.cc
        google/protobuf/compiler/c/c_helpers.cc
        google/protobuf/compiler/c/c_enum.cc
        google/protobuf/compiler/c/c_enum_field.cc
        google/protobuf/compiler/c/c_string_field.cc
        google/protobuf/compiler/c/c_primitive_field.cc
        google/protobuf/compiler/c/c_extension.cc
        google/protobuf/compiler/c/c_file.cc
        google/protobuf/compiler/c/c_field.cc
        google/protobuf/compiler/c/c_message.cc
        google/protobuf/compiler/c/c_generator.cc
        google/protobuf/compiler/c/c_message_field.cc
        google/protobuf/compiler/c/main.cc
        google/protobuf/compiler/c/c_bytes_field.cc)
    add_executable(protoc-c-ng ${PROTOC_C_SOURCES})
    set_target_properties(protoc-c PROPERTIES OUTPUT_NAME protoc-c-ng)
	target_include_directories(protoc-c PRIVATE ${PROTOBUF_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR})
    target_link_libraries(protoc-c PUBLIC ${PROTOBUF_LIBRARY}
        ${PROTOBUF_PROTOC_LIBRARY})
    if ( UNIX )
        find_package(Threads REQUIRED)
        target_link_libraries(protoc-c PUBLIC ${CMAKE_THREAD_LIBS_INIT})
    endif()
    install(TARGETS protoc-c DESTINATION ${PROTOBUF_C_INSTALL_BINDIR})
endif()

include(CheckIncludeFiles)
check_include_files("alloca.h" HAVE_ALLOCA_H)
check_include_files("malloc.h" HAVE_MALLOC_H)
check_include_files("sys/poll.h" HAVE_SYS_POLL_H)
check_include_files("sys/select.h" HAVE_SYS_SELECT_H)
check_include_files("inttypes.h" HAVE_INTTYPES_H)
check_include_files("sys/uio.h" HAVE_SYS_UIO_H)
check_include_files("unistd.h" HAVE_UNISTD_H)
check_include_files("io.h" HAVE_IO_H)
include(TestBigEndian)
test_big_endian(IS_BIG_ENDIAN)
if ( NOT IS_BIG_ENDIAN )
    set(IS_LITTLE_ENDIAN 1)
endif()
#add_definitions(-DHAVE_PROTOBUF_C_CONFIG_H=1)
#configure_file(google/protobuf-c/protobuf-c-config.h.in google/protobuf-c/protobuf-c-config.h)

set(BUILD_PROTOBUF_C_STATIC_LIB ON
    CACHE BOOL "build static protobuf-c library")
set(BUILD_PROTOBUF_C_SHARED_LIB OFF
    CACHE BOOL "build shared protobuf-c library")

if ( WITH_PROTOBUF_C_RPC )
    set(PROTOBUF_C_SOURCES
        google/protobuf-c/protobuf-c-dispatch.c
        google/protobuf-c/protobuf-c-data-buffer.c
        google/protobuf-c/protobuf-c-rpc.c
        google/protobuf-c/protobuf-c.c)
else()
    set(PROTOBUF_C_SOURCES
        google/protobuf-c/protobuf-c-data-buffer.c
        google/protobuf-c/protobuf-c.c)
endif()

if ( BUILD_PROTOBUF_C_STATIC_LIB )
    add_library(protobuf-c STATIC ${PROTOBUF_C_SOURCES})
    install(TARGETS protobuf-c
        ARCHIVE DESTINATION ${PROTOBUF_C_INSTALL_LIBDIR})
    set_target_properties(protobuf-c PROPERTIES OUTPUT_NAME protobuf-c-ng)

    get_target_property(DIRECTORY protobuf-c ARCHIVE_OUTPUT_DIRECTORY) 
    set_target_properties(protobuf-c PROPERTIES ARCHIVE_OUTPUT_DIRECTORY
        ${DIRECTORY}/static)
elseif ( BUILD_PROTOBUF_C_SHARED_LIB )
    add_library(protobuf-c SHARED ${PROTOBUF_C_SOURCES})
    install(TARGETS protobuf-c
        RUNTIME DESTINATION ${PROTOBUF_C_INSTALL_BINDIR}
        LIBRARY DESTINATION ${PROTOBUF_C_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${PROTOBUF_C_INSTALL_LIBDIR}/dllimport)
    set_target_properties(protobuf-c PROPERTIES OUTPUT_NAME protobuf-c-ng)

    set_target_properties(protobuf-c PROPERTIES VERSION
        ${PROTOBUF_C_VERSION})
    get_target_property(DIRECTORY protobuf-c ARCHIVE_OUTPUT_DIRECTORY) 
    set_target_properties(protobuf-c PROPERTIES ARCHIVE_OUTPUT_DIRECTORY
        ${DIRECTORY}/shared)
    get_target_property(DIRECTORY protobuf-c LIBRARY_OUTPUT_DIRECTORY) 
    set_target_properties(protobuf-c PROPERTIES LIBRARY_OUTPUT_DIRECTORY
        ${DIRECTORY}/shared)
    if ( WIN32 )
        set_target_properties(protobuf-c PROPERTIES COMPILE_DEFINITIONS
            "PROTOBUF_C_USE_SHARED_LIB;PROTOBUF_C_EXPORT")
    endif()
endif()

target_include_directories(protobuf-c 
	PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/google
	PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/google/protobuf-c
	)

if ( BUILD_PROTOBUF_C_STATIC_LIB OR BUILD_PROTOBUF_C_SHARED_LIB )
    if ( WITH_PROTOBUF_C_RPC  )
        set(PROTOBUF_C_PUBLIC_HEADERS
            google/protobuf-c/protobuf-c.h
            google/protobuf-c/protobuf-c-private.h
            google/protobuf-c/protobuf-c-dispatch.h
            google/protobuf-c/protobuf-c-rpc.h)
    else()
        set(PROTOBUF_C_PUBLIC_HEADERS
            google/protobuf-c/protobuf-c.h
            google/protobuf-c/protobuf-c-private.h)
    endif()
    install(FILES ${PROTOBUF_C_PUBLIC_HEADERS} DESTINATION
        ${PROTOBUF_C_INSTALL_INCLUDEDIR})
endif()

if ( WITH_PROTOBUF_C_TEST )
	add_subdirectory(test)
endif()

