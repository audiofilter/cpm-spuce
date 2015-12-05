CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME spuce)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_QUIET ERROR_QUIET)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()


# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

# Include CPM modules or externals here (with CPM_AddModule).
CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# Spuce
#------------------------------------------------------------------------------
# 
set(SPUCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/spuce)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${SPUCE_DIR}
  GIT_REPOSITORY "https://github.com/audiofilter/spuce.git"
  GIT_TAG 0.4.1
  USE_CACHING TRUE
  )

SET(FILT_SRCS  
	${SPUCE_DIR}/filters/butterworth_iir.cpp
	${SPUCE_DIR}/filters/chebyshev_iir.cpp
	${SPUCE_DIR}/filters/chebyshev2_iir.cpp
	${SPUCE_DIR}/filters/elliptic_iir.cpp
	${SPUCE_DIR}/filters/find_roots.cpp
	${SPUCE_DIR}/filters/fir_coeff.cpp
	${SPUCE_DIR}/filters/iir_coeff.cpp
  ${SPUCE_DIR}/filters/design_iir.cpp
	# FIR stuff
  ${SPUCE_DIR}/filters/design_fir.cpp
	${SPUCE_DIR}/filters/gaussian_fir.cpp
	${SPUCE_DIR}/filters/butterworth_fir.cpp
	${SPUCE_DIR}/filters/raised_cosine_imp.cpp
	${SPUCE_DIR}/filters/root_raised_cosine_imp.cpp
	${SPUCE_DIR}/filters/remez_estimate.cpp
	${SPUCE_DIR}/filters/remez_fir.cpp
	${SPUCE_DIR}/filters/sinc_fir.cpp
	${SPUCE_DIR}/filters/sinc_helper.cpp
	${SPUCE_DIR}/filters/create_remez_lpfir.cpp
	${SPUCE_DIR}/filters/fir_inv_dft.cpp
	${SPUCE_DIR}/filters/transform_fir.cpp
	# Windows
	${SPUCE_DIR}/filters/window.cpp
  ${SPUCE_DIR}/filters/design_window.cpp
	# Other filters
	${SPUCE_DIR}/filters/butterworth_allpass.cpp
	${SPUCE_DIR}/filters/elliptic_allpass.cpp
  ${SPUCE_DIR}/filters/shelf_allpass1.cpp
  ${SPUCE_DIR}/filters/farrow_upsampler.cpp
  #
  ${SPUCE_DIR}/filters/calculate_decimator_taps.cpp
)

include_directories(${SPUCE_DIR})
add_library(${CPM_LIB_TARGET_NAME} STATIC ${SPUCE_SRCS})
target_link_libraries(${CPM_LIB_TARGET_NAME} ${CPM_LIBRARIES})
set_property(TARGET ${CPM_LIB_TARGET_NAME} PROPERTY CXX_STANDARD 11)

CPM_ExportAdditionalIncludeDir("${SPUCE_DIR}")

