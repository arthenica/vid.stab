cmake_minimum_required (VERSION 2.6)
project (vid.stab.transcode)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/../CMakeModules/")

# set your transcode path here!
set(TRANSCODE_ROOT ../../transcode)

# Default to release builds if no explicit build type specified.
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "Release")
endif()
if (NOT MSVC)
	add_definitions(-Wall -Wno-pointer-sign -DTRANSCODE -std=gnu99)
	if(SSE2_FOUND)
		add_definitions( -DUSE_SSE2 -msse2 -ffast-math )
	endif()
endif ()


# here we should check for SSE2
# our  -DUSE_SSE2_ASM code does not work with fpic


# Make sure the compiler can find include files from transcode
include_directories (../src ${TRANSCODE_ROOT}/src ${TRANSCODE_ROOT}/ )

# Create the stabilize and transform libraries
add_library (filter_transform SHARED filter_transform.c
  ../src/transformtype.c ../src/libvidstab.c ../src/transform.c
  ../src/transformfixedpoint.c  ../src/vsvector.c ../src/serialize.c ../src/frameinfo.c
	../src/localmotion2transform.c)
add_library (filter_stabilize SHARED filter_stabilize.c
  ../src/transformtype.c ../src/libvidstab.c ../src/motiondetect.c
	../src/motiondetect_opt.c ../src/localmotion2transform.c
  ../src/boxblur.c ../src/vsvector.c ../src/serialize.c  ../src/frameinfo.c)
add_library (filter_deshake SHARED filter_deshake.c
  ../src/transformtype.c ../src/libvidstab.c ../src/motiondetect.c
  ../src/boxblur.c ../src/transform.c ../src/motiondetect_opt.c
  ../src/transformfixedpoint.c ../src/vsvector.c ../src/serialize.c ../src/frameinfo.c
	../src/localmotion2transform.c)


set_target_properties(filter_stabilize PROPERTIES  PREFIX "" )
set_target_properties(filter_transform PROPERTIES  PREFIX "" )
set_target_properties(filter_deshake   PROPERTIES  PREFIX "" )

target_link_libraries(filter_stabilize m )
target_link_libraries(filter_transform m )
target_link_libraries(filter_deshake   m )

if(USE_OMP)
target_link_libraries(filter_stabilize gomp )
target_link_libraries(filter_transform gomp )
target_link_libraries(filter_deshake   gomp )
endif()
