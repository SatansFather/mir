
cmake_minimum_required (VERSION 3.8)

#set(CMAKE_C_COMPILER "clang-cl")

project ("MIR")

if(NOT DEFINED CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()

#set(CMAKE_VERBOSE_MAKEFILE ON)

if(NOT WIN32)
  add_compile_options(
    $<$<CONFIG:RELEASE>:-O3>
    $<$<CONFIG:RelWithDebInfo>:-O3>
  )
  add_compile_options(-std=gnu11 -Wno-abi -fsigned-char -fPIC)
endif()

include(CheckCCompilerFlag)
if(CMAKE_COMPILER_IS_GNUCC)
  check_c_compiler_flag(-fno-tree-sra NO_TREE_SRA)
  if (NO_TREE_SRA)
    add_compile_options(-fno-tree-sra)
  endif()
  check_c_compiler_flag(-fno-ipa-cp-clone NO_IPA_CP_CLONE)
  if (NO_IPA_CP_CLONE)
    add_compile_options(-fno-ipa-cp-clone)
  endif()
endif()

include(FindThreads)
if(Threads_FOUND)
  link_libraries(${CMAKE_THREAD_LIBS_INIT})
endif()

message("C compiler flags: ${CMAKE_C_FLAGS}")

option(BUILD_TESTING "" ON)
include(CTest)

add_library(mir OBJECT mir.c mir-gen.c c2mir/c2mir.c mir.h mir-gen.h c2mir/c2mir.h)
target_include_directories(mir PRIVATE ${PROJECT_SOURCE_DIR})
if(Threads_FOUND)
  target_compile_definitions(mir PUBLIC "MIR_PARALLEL_GEN")
endif()
if(UNIX)
	target_link_libraries(mir m)
endif()

# ------------------ LIBMIR -----------------------
add_library(mir_static STATIC)
target_link_libraries(mir_static PRIVATE mir)

# ------------------ LIBMIR SO --------------------
add_library(mir_shared SHARED)
if(WIN32)
  set_target_properties(mir_shared PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()
target_link_libraries(mir_shared PRIVATE mir)

# ------------------ C2M --------------------------
add_executable (c2m "c2mir/c2mir-driver.c")
target_include_directories(c2m PRIVATE ${PROJECT_SOURCE_DIR})
target_link_libraries(c2m mir ${CMAKE_DL_LIBS} )

set(MIR_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR} CACHE PATH "mir include" FORCE)
