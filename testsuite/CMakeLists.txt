set(CMAKE_CXX_STANDARD 23)

file(COPY
    ${CMAKE_CURRENT_SOURCE_DIR}/alice.md.decrypted
    ${CMAKE_CURRENT_SOURCE_DIR}/alice.md.encrypted
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/)

enable_testing()

add_executable(testsuite)

if("${CMAKE_SYSTEM_NAME}" STREQUAL "Android")
    if("${CMAKE_ANDROID_ARCH_ABI}" STREQUAL "arm64-v8a")
        set(SOURCES arm_v7_main.cxx)
    elseif("${CMAKE_ANDROID_ARCH_ABI}" STREQUAL "armeabi-v7a")
        set(OPTIONS -mfpu=neon)
        set(SOURCES arm_v7_main.cxx)
    elseif("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "x86_64")
        set(OPTIONS -msse3 -maes)
        set(SOURCES x86_64_main.cxx)
    else()
        set(SOURCES main.cxx)
    endif()
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS")
    if("${CMAKE_OSX_ARCHITECTURES}" STREQUAL "arm64")
        set(SOURCES aarch64_main.cxx)
    elseif("${CMAKE_OSX_ARCHITECTURES}" STREQUAL "x86_64")
        set(OPTIONS -msse3 -maes)
        set(SOURCES x86_64_main.cxx)
    endif()
    set_target_properties(testsuite PROPERTIES
        XCODE_ATTRIBUTE_CODE_SIGNING_ALLOWED "NO"
        XCODE_ATTRIBUTE_ENABLE_BITCODE "NO"
        MACOSX_BUNDLE_BUNDLE_NAME mimicssl-aes128-cbc-decrypt-testsuite
        MACOSX_BUNDLE_GUI_IDENTIFIER mimicssl-aes128-cbc-decrypt-testsuite
        MACOSX_BUNDLE_BUNDLE_VERSION 1.0
        MACOSX_BUNDLE_SHORT_VERSION_STRING 1.0
        MACOSX_BUNDLE_LONG_VERSION_STRING 1.0)
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin"
        AND "${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "arm64")
    set(SOURCES aarch64_main.cxx)
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows"
        AND "${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "AMD64")
    set(SOURCES x86_64_main.cxx)
elseif("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "x86_64")
    set(OPTIONS -msse3 -maes)
    set(SOURCES x86_64_main.cxx)
else()
    set(SOURCES main.cxx)
endif()

if("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS")
    set(DEFINES IOS_PROCESS_DELAY_WORKAROUND=1)
else()
    set(DEFINES IOS_PROCESS_DELAY_WORKAROUND=0)
endif()

target_compile_options(testsuite PUBLIC ${OPTIONS})
target_compile_definitions(testsuite PUBLIC ${DEFINES})
target_sources(testsuite PUBLIC
    ${SOURCES}
    expect.hxx expect_fallback.hxx)
target_include_directories(testsuite PRIVATE
    mimicssl-aes128-cbc-decrypt
    ${CMAKE_SOURCE_DIR}/libmimicssl-aes128-cbc-decrypt/src
    ${CMAKE_SOURCE_DIR}/lighter/include)

target_link_libraries(testsuite mimicssl-aes128-cbc-decrypt)

include(GoogleTest)
gtest_discover_tests(testsuite
    DISCOVERY_TIMEOUT 500
    DISCOVERY_MODE PRE_TEST)
