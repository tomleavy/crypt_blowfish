cmake_minimum_required(VERSION 3.10 FATAL_ERROR)

include(GNUInstallDirs)

project(libbcrypt VERSION 1.3 LANGUAGES C)

if(NOT IOS)
    enable_testing()
endif()

set(BcryptSources crypt_blowfish.c crypt_gensalt.c wrapper.c)

Message("${CMAKE_SYSTEM_PROCESSOR}")

if("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "i686" OR (WIN32 AND NOT "${CMAKE_GENERATOR}" MATCHES "(Win64|IA64)"))
    
    if (NOT WIN32)
        enable_language(ASM)
    else()
        enable_language(ASM_MASM)
    endif()

    set(BcryptSources ${BcryptSources} x86.S)
    Message("Bcrypt: Enabling x86 ASM")
endif()

add_library(bcrypt ${BcryptSources})

set(BcryptCompileFlags "-Wall" "-fomit-frame-pointer" "-funroll-loops")

string (REPLACE ";" " " BCRYPT_COMPILE_FLAGS "${BcryptCompileFlags}")

set_target_properties(bcrypt PROPERTIES COMPILE_FLAGS "${BCRYPT_COMPILE_FLAGS}")

install(TARGETS bcrypt EXPORT BcryptConfig
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR})

install(EXPORT BcryptConfig DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake)

file(GLOB Headers *.h)

install(FILES ${Headers} DESTINATION include/libbcrypt)

export(TARGETS bcrypt FILE BcryptConfig.cmake)

if (NOT IOS)
    add_executable(crypt_test wrapper.c)

    target_link_libraries(crypt_test bcrypt)

    set_target_properties(crypt_test PROPERTIES COMPILE_FLAGS "${BCRYPT_COMPILE_FLAGS} -DTEST")

    add_test(crypt_test crypt_test)
endif ()
