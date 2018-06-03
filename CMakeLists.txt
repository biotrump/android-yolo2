cmake_minimum_required(VERSION 3.0.0)
project(YOLO-NDK C CXX)

##
## PROJECT
## name and version
##
project(YOLO-NDK VERSION 2 LANGUAGES CXX)

message("YOLO-NDK_SOURCE_DIR = ${YOLO-NDK_SOURCE_DIR}")
message("YOLO-NDK_BINARY_DIR = ${YOLO-NDK_BINARY_DIR}")

# --------------------------------------------------
# Organize output files.  On Windows this also keeps .dll files next
# to the .exe files that need them, making tests easy to run.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${YOLO-NDK_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${YOLO-NDK_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${YOLO-NDK_BINARY_DIR}/lib)

##
## INCLUDE
##
##
#include(ExternalProject)

##
## OPTIONS
##

option(DEBUG "Compile with debugging information" OFF)

if(DEBUG)
	set(CMAKE_BUILD_TYPE "Debug")
endif()
message("***CMAKE_BUILD_TYPE = ${CMAKE_BUILD_TYPE}")

#option(JSON_BuildTests "Build the unit tests when BUILD_TESTING is enabled." ON)
#option(JSON_MultipleHeaders "Use non-amalgamated version of the library." OFF)

##
## CONFIGURATION
##


#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unknown-pragmas -Wfatal-errors -fPIC")
#set(CMAKE_LD_FLAGS "${CMAKE_LD_FLAGS} -lm -pthread -fopenmp")

if("${CMAKE_BUILD_TYPE}" MATCHES "Debug")
  add_definitions(-DDEBUG)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -O0 -fPIC")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -g -O0 -fPIC -D_POSIX_C_SOURCE=200809")
else()
  add_definitions(-DARMA_NO_DEBUG)
  add_definitions(-DNDEBUG)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3 -fPIC")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -O3 -fPIC -D_POSIX_C_SOURCE=200809")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstrict-aliasing -mfpu=neon-vfpv4 -funsafe-math-optimizations -ftree-vectorize")
endif()

option(PROFILE "Compile with profiling information" OFF)
# Profiling CFLAGS.  Turn profiling information on.
if(PROFILE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pg")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pg")
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unknown-pragmas -Wfatal-errors")
#neno opt
#openmp opt -fopenmp 
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -fexceptions -Dlinux -fsigned-char")

#set(CMAKE_LD_FLAGS "${CMAKE_LD_FLAGS} -lm -pthread -fopenmp")

# --------------------------------------------------
#The following libs are all built in working folders,
#so the libs' path are not in the toolchain path.
#The path is in the host path, not in the tool chain path.
#disable searching toolchain path.
# --------------------------------------------------
# switch searching path to host, not NDK or arm gnu
# If set to NEVER, then the roots in CMAKE_FIND_ROOT_PATH will be ignored
# and only the host system root will be used
# If set to ONLY, then only the roots in CMAKE_FIND_ROOT_PATH will be searched.
# host path is NOT searched.
# --------------------------------------------------
if(ANDROID_NDK OR ARM_LINUX_SYSROOT)
set( CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER )
set( CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY )
set( CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY )
endif()
# --------------------------------------------------
include_directories(
	${YOLO-NDK_SOURCE_DIR}/darknet/src
	${YOLO-NDK_SOURCE_DIR}/darknet/include
)
# --------------------------------------------------
#openMP to avoid link error : omp_xxx error!
# undefined reference to `GOMP_loop_dynamic_start
find_package(OpenMP)
if(OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	add_definitions(-DENABLE_OPENMP)
endif()
# --------------------------------------------------
if(ANDROID_NDK)
find_package (Threads REQUIRED)
endif()
# --------------------------------------------------
# --------------------------------------------------
#Cmakefilelist : some source files are not used
file(GLOB darknet_files "darknet/src/*.c")
# --------------------------------------------------
# --------------------------------------------------
add_library( # Sets the name of the library.
             darknetlib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             ${darknet_files}
             )
target_link_libraries(darknetlib
	log
	m
	stdc++
	)
# --------------------------------------------------
if(JNI_ENABLED)
add_library( # Sets the name of the library.
             darknetlib-jni

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
#             ${DARKNET_SRC_LISTS}
             $<TARGET_OBJECTS:darknet_filesV2>
             jni/darknetlib.c
             )

target_link_libraries(darknetlib-jni
		log
		m
		stdc++)
endif()

#
#add_dependencies(darknet darknetlib)
#example 
add_executable(darknet
	darknet/examples/attention.c
	darknet/examples/art.c
	darknet/examples/cifar.c
	darknet/examples/darknet.c
	darknet/examples/lsd.c
	darknet/examples/rnn.c
	darknet/examples/super.c
#	darknet/examples/voxel.c
#	darknet/examples/attention.c
	darknet/examples/classifier.c
	darknet/examples/detector.c
#	darknet/examples/dice.c
	darknet/examples/nightmare.c
#	darknet/examples/rnn_vid.c
# 	darknet/examples/swag.c
#	darknet/examples/writing.c
	darknet/examples/captcha.c
	darknet/examples/coco.c
	darknet/examples/go.c
	darknet/examples/regressor.c
	darknet/examples/segmenter.c
	darknet/examples/tag.c
	darknet/examples/yolo.c
	)
	
target_link_libraries(darknet
	darknetlib
	log
	m
	stdc++
	)
#
add_executable(test
	darknet/examples/test.c
	)
	
target_link_libraries(test
	darknetlib
	log
	m
	stdc++
	)