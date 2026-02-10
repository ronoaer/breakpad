cmake_minimum_required(VERSION 3.16)

project(breakpad LANGUAGES C CXX ASM)

set(BREAKPAD_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")

set(BREAKPAD_SOURCES
    ${BREAKPAD_SRC_DIR}/common/string_conversion.cc
    ${BREAKPAD_SRC_DIR}/common/convert_UTF.cc
    ${BREAKPAD_SRC_DIR}/common/md5.cc
)

if(WIN32)
    list(APPEND BREAKPAD_SOURCES
        ${BREAKPAD_SRC_DIR}/client/windows/handler/exception_handler.cc
        ${BREAKPAD_SRC_DIR}/client/windows/crash_generation/crash_generation_client.cc
        ${BREAKPAD_SRC_DIR}/common/windows/guid_string.cc
        ${BREAKPAD_SRC_DIR}/common/windows/http_upload.cc
        ${BREAKPAD_SRC_DIR}/client/windows/sender/crash_report_sender.cc
    )
    set(PLATFORM_LIBS wininet winhttp)
    set(PLATFORM_DEFS _CRT_SECURE_NO_WARNINGS NOMINMAX UNICODE _UNICODE)

elseif(APPLE)
    enable_language(OBJCXX)

    list(APPEND BREAKPAD_SOURCES
        ${BREAKPAD_SRC_DIR}/client/mac/handler/exception_handler.cc
        ${BREAKPAD_SRC_DIR}/client/mac/handler/minidump_generator.cc
        ${BREAKPAD_SRC_DIR}/client/mac/handler/dynamic_images.cc
        ${BREAKPAD_SRC_DIR}/client/mac/handler/breakpad_nlist_64.cc
        ${BREAKPAD_SRC_DIR}/client/mac/crash_generation/crash_generation_client.cc
        ${BREAKPAD_SRC_DIR}/common/mac/file_id.cc
        ${BREAKPAD_SRC_DIR}/common/mac/macho_id.cc
        ${BREAKPAD_SRC_DIR}/common/mac/macho_utilities.cc
        ${BREAKPAD_SRC_DIR}/common/mac/macho_walker.cc
        ${BREAKPAD_SRC_DIR}/common/mac/string_utilities.cc
        ${BREAKPAD_SRC_DIR}/common/mac/arch_utilities.cc
        ${BREAKPAD_SRC_DIR}/common/mac/bootstrap_compat.cc
        ${BREAKPAD_SRC_DIR}/common/mac/MachIPC.mm
        ${BREAKPAD_SRC_DIR}/client/minidump_file_writer.cc
    )
    set(PLATFORM_LIBS "-framework CoreFoundation")

    add_compile_options("-includeAvailabilityMacros.h")
    set(PLATFORM_DEFS
        MAC_OS_X_VERSION_MIN_REQUIRED=101500
        MAC_OS_X_VERSION_MAX_ALLOWED=101500
    )

else()
    list(APPEND BREAKPAD_SOURCES
        ${BREAKPAD_SRC_DIR}/client/linux/handler/exception_handler.cc
        ${BREAKPAD_SRC_DIR}/client/linux/handler/minidump_descriptor.cc
        ${BREAKPAD_SRC_DIR}/client/linux/log/log.cc
        ${BREAKPAD_SRC_DIR}/client/linux/crash_generation/crash_generation_client.cc
        ${BREAKPAD_SRC_DIR}/common/linux/guid_creator.cc
        ${BREAKPAD_SRC_DIR}/common/linux/elfutils.cc
        ${BREAKPAD_SRC_DIR}/common/linux/file_id.cc
        ${BREAKPAD_SRC_DIR}/common/linux/linux_libc_support.cc
        ${BREAKPAD_SRC_DIR}/common/linux/memory_mapped_file.cc
        ${BREAKPAD_SRC_DIR}/common/linux/safe_readlink.cc
        ${BREAKPAD_SRC_DIR}/client/linux/minidump_writer/minidump_writer.cc
        ${BREAKPAD_SRC_DIR}/client/linux/minidump_writer/linux_dumper.cc
        ${BREAKPAD_SRC_DIR}/client/linux/minidump_writer/linux_ptrace_dumper.cc
        ${BREAKPAD_SRC_DIR}/client/linux/minidump_writer/pe_file.cc
        ${BREAKPAD_SRC_DIR}/client/linux/microdump_writer/microdump_writer.cc
        ${BREAKPAD_SRC_DIR}/client/linux/dump_writer_common/thread_info.cc
        ${BREAKPAD_SRC_DIR}/client/linux/dump_writer_common/ucontext_reader.cc
        ${BREAKPAD_SRC_DIR}/common/linux/breakpad_getcontext.S
        ${BREAKPAD_SRC_DIR}/client/minidump_file_writer.cc
    )
    find_package(Threads REQUIRED)
    set(PLATFORM_LIBS Threads::Threads)
    include_directories(${BREAKPAD_SRC_DIR}/third_party/lss)
endif()

add_library(breakpad STATIC ${BREAKPAD_SOURCES})

target_compile_definitions(breakpad PRIVATE ${PLATFORM_DEFS})

target_include_directories(breakpad
    PUBLIC
        $<BUILD_INTERFACE:${BREAKPAD_SRC_DIR}>
)

target_link_libraries(breakpad PUBLIC ${PLATFORM_LIBS})

target_compile_features(breakpad PUBLIC cxx_std_17)

add_library(Breakpad::Client ALIAS breakpad)
add_library(breakpad_client ALIAS breakpad)
