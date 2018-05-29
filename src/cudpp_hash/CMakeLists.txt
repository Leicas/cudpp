##############################################################################
#
# CUDA Data Parallel Primitives (CUDPP) CMake file
#
###############################################################################

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/../../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../../src/cudpp
  ${CMAKE_CURRENT_SOURCE_DIR}
  )

set(CCFILES
  cudpp_hash.cpp
  debugging.cpp
  hash_compacting.cpp
  hash_multivalue.cpp
  hash_table.cpp
  mt19937ar.cpp
  )

set (HFILES
  debugging.h
  definitions.h
  hash_compacting.h
  hash_functions.h
  hash_multivalue.h
  hash_table.h
  mt19937ar.h
  )

set (CUHFILES
  hash_table.cuh
  )

set(CUFILES
  debugging.cu
  hash_compacting.cu
  hash_multivalue.cu
  hash_table.cu
  hash_functions.cu
  )

set(HFILES_PUBLIC
  ../../include/cudpp_hash.h
  )

source_group("CUDA Source Files" FILES ${CUFILES})
source_group("CUDA Header Files" FILES ${CUHFILES})

set(GENCODE_SM30 -gencode=arch=compute_30,code=sm_30 -gencode=arch=compute_30,code=compute_30)
set(GENCODE_SM35 -gencode=arch=compute_35,code=sm_35 -gencode=arch=compute_35,code=compute_35)
set(GENCODE_SM50 -gencode=arch=compute_50,code=sm_50 -gencode=arch=compute_50,code=compute_50)
set(GENCODE_SM52 -gencode=arch=compute_52,code=sm_52 -gencode=arch=compute_52,code=compute_52)
set(GENCODE_SM60 -gencode=arch=compute_60,code=sm_60 -gencode=arch=compute_60,code=compute_60)
set(GENCODE_SM61 -gencode=arch=compute_61,code=sm_61 -gencode=arch=compute_61,code=compute_61)

option(CUDPP_GENCODE_SM30
       "ON to generate code for Compute Capability 3.0 devices (e.g. Tesla K10)"
       ON)

option(CUDPP_GENCODE_SM35
       "ON to generate code for Compute Capability 3.5 devices (e.g. Tesla K20)"
       OFF)

option(CUDPP_GENCODE_SM50
       "ON to generate code for Compute Capability 5.0 devices (e.g. GeForce GTX 750)"
       OFF)

option(CUDPP_GENCODE_SM52
       "ON to generate code for Compute Capability 5.2 devices (e.g. GeForce GTX 950)"
       OFF)

option(CUDPP_GENCODE_SM60
       "ON to generate code for Compute Capability 6.0 devices (e.g. Tesla P100)"
       OFF)

option(CUDPP_GENCODE_SM61
       "ON to generate code for Compute Capability 6.1 devices (e.g. GeForce GTX 1050)"
       OFF)

if (CUDPP_GENCODE_SM30)
  set(GENCODE ${GENCODE} ${GENCODE_SM30})
endif(CUDPP_GENCODE_SM30)

if (CUDPP_GENCODE_SM35)
  set(GENCODE ${GENCODE} ${GENCODE_SM35})
endif(CUDPP_GENCODE_SM35)

if (CUDPP_GENCODE_SM50)
  set(GENCODE ${GENCODE} ${GENCODE_SM50})
endif(CUDPP_GENCODE_SM50)

if (CUDPP_GENCODE_SM52)
  set(GENCODE ${GENCODE} ${GENCODE_SM52})
endif(CUDPP_GENCODE_SM52)

if (CUDPP_GENCODE_SM60)
  set(GENCODE ${GENCODE} ${GENCODE_SM60})
endif(CUDPP_GENCODE_SM60)

if (CUDPP_GENCODE_SM61)
  set(GENCODE ${GENCODE} ${GENCODE_SM61})
endif(CUDPP_GENCODE_SM61)

if (CUDA_VERBOSE_PTXAS)
  set(VERBOSE_PTXAS --ptxas-options=-v)
endif (CUDA_VERBOSE_PTXAS)

if (WIN32)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
endif (WIN32)

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  exec_program(uname ARGS -v  OUTPUT_VARIABLE DARWIN_VERSION)
  string(REGEX MATCH "[0-9]+" DARWIN_VERSION ${DARWIN_VERSION})
  # message(STATUS "Detected Darwin kernel version: ${DARWIN_VERSION}")
  if ((DARWIN_VERSION GREATER 13) OR (DARWIN_VERSION EQUAL 13))
    set(APPLE_MAVERICKS_OR_LATER 1 INTERNAL)
    set(CMAKE_CXX_FLAGS -stdlib=libstdc++)
    set(CMAKE_C_FLAGS -stdlib=libstdc++)
    if(CUDA_VERSION_MAJOR EQUAL 6 AND CUDA_VERSION_MINOR LESS 5)
      set(CUDA_6_0 1)
    endif(CUDA_VERSION_MAJOR EQUAL 6 AND CUDA_VERSION_MINOR LESS 5)
    if(CUDA_VERSION_MAJOR LESS 6 OR CUDA_6_0)
      if(NOT CUDA_NVCC_FLAGS MATCHES "ccbin")
        set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -ccbin;/usr/bin/clang)
      endif(NOT CUDA_NVCC_FLAGS MATCHES "ccbin")
    endif(CUDA_VERSION_MAJOR LESS 6 OR CUDA_6_0)
  endif ((DARWIN_VERSION GREATER 13) OR (DARWIN_VERSION EQUAL 13))
  # current CUDA on Mavericks uses libstdc++, not libc++
  # https://github.com/cudpp/cudpp/wiki/BuildingCUDPPWithMavericks
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

cuda_add_library(cudpp_hash ${LIB_TYPE}
  ${CCFILES}
  ${HFILES}
  ${CUHFILES}
  ${HFILES_PUBLIC}
  ${CUFILES}
  OPTIONS ${GENCODE} ${VERBOSE_PTXAS}
  )

target_link_libraries(cudpp_hash cudpp)

install(FILES ${HFILES_PUBLIC}
  DESTINATION include
  )

install(TARGETS cudpp_hash
  DESTINATION lib
  EXPORT cudpp-targets
  )

install(EXPORT cudpp-targets
  DESTINATION lib
  )
