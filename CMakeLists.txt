cmake_minimum_required(VERSION 3.0)
set(CMP0048 NEW)

project("GOOC Compiler"
  VERSION 1)
set(PROJECT_URL "https://github.com/mandude/gooc")
set(PROJECT_BUGREPORT "https://github.com/mandude/gooc/issues")

include(cmake/CheckStructPacking.cmake)
include(CheckTypeSize)
include(CheckIncludeFile)
include(CheckSymbolExists)
include(CMakePushCheckState)

check_packing_gnuc_attribute()
if(NOT ${HAVE_PACKING_GNUC_ATTRIBUTE})
    check_packing_pragma_push()
    if(NOT ${HAVE_PACKING_PRAGMA_PUSH})
        message(SEND_ERROR "No structure packing.")
    endif()
endif()

check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
if(${HAVE_SYS_TYPES_H})
  set(CMAKE_EXTRA_INCLUDE_FILES "sys/types.h")
  check_type_size(off_t OFF_T BUILTIN_TYPES_ONLY)
  check_type_size(ssize_t SSIZE_T BUILTIN_TYPES_ONLY)
  cmake_reset_check_state()
endif()
if(NOT ${HAVE_SSIZE_T})
  cmake_reset_check_state()
  set(CMAKE_EXTRA_INCLUDE_FILES "basetsd.h")
  check_type_size(SSIZE_T SSIZE_T_BASETSD BUILTIN_TYPES_ONLY)
  cmake_reset_check_state()
endif()
cmake_pop_check_state()

check_include_file("stdint.h" HAVE_STDINT_H)
check_include_file("unistd.h" HAVE_UNISTD_H)

check_symbol_exists("mmap" "sys/mman.h" HAVE_MMAP)
check_symbol_exists("scandir" "dirent.h" HAVE_SCANDIR)
check_symbol_exists("fstat" "sys/stat.h" HAVE_FSTAT)
check_symbol_exists("fileno" "stdio.h" HAVE_FILENO)

check_symbol_exists("getc_unlocked" "stdio.h" HAVE_GETC_UNLOCKED)
if(${HAVE_GETC_UNLOCKED})
  check_symbol_exists("fread_unlocked" "stdio.h" HAVE_FREAD_UNLOCKED)
  if(${HAVE_FREAD_UNLOCKED})
    check_symbol_exists("feof_unlocked" "stdio.h" HAVE_FEOF_UNLOCKED)
  endif()
else()
  check_symbol_exists("_getc_nolock" "stdio.h" HAVE__GETC_NOLOCK)
  if(${HAVE__GETC_NOLOCK})
    check_symbol_exists("_fread_nolock" "stdio.h" HAVE__FREAD_NOLOCK)
  endif()
endif()

set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/lib)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

if(UNIX)
  option(CONTRIB_UTHDAT "Build midnight commander plugin" OFF)
endif()
if(WIN32)
  option(CONTRIB_WCTHDAT "Build total commander plugin" OFF)
endif()

if(MSVC)
  # Defining these here rather than in config.h.in also applies them to the
  # zlib-ng and libpng build.
  # Character set options were introduced in Visual Studio 2015 Update 2
  add_compile_options(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE /source-charset:utf-8 /execution-charset:utf-8)
else()
  add_compile_options(-D_GNU_SOURCE -std=c99 -finput-charset=utf-8 -fexec-charset=utf-8)
endif()

function(link_setargv target)
  if(MSVC)
    set_target_properties(${target} PROPERTIES LINK_FLAGS "setargv.obj")
  endif()
endfunction()

if(MSVC)
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT /GL /Ox /Ot")
  set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /INCREMENTAL:NO /OPT:REF /LTCG")
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /INCREMENTAL:NO /OPT:REF /LTCG")
  set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} /MT /GL /Ox /Ot")
  set(CMAKE_SHARED_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_SHARED_LINKER_FLAGS_RELWITHDEBINFO} /INCREMENTAL:NO /OPT:REF /LTCG")
  set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO} /INCREMENTAL:NO /OPT:REF /LTCG")
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")
endif()

set(BISON_FLAGS "" CACHE STRING "Flags used by the BISON parser generator during all build types.")

include_directories(${CMAKE_CURRENT_BINARY_DIR})
add_subdirectory(extlib)
add_subdirectory(util)
include_directories(util extlib/thtypes)
add_subdirectory(thecl)
add_subdirectory(thtk)
add_subdirectory(contrib)

configure_file(config.h.in config.h)
