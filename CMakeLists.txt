# cmake -DCMAKE_BUILD_TYPE=Debug ..
# cmake -DCMAKE_BUILD_TYPE=Release ..
cmake_minimum_required(VERSION 2.8)
project(bento4)

option(BENTO4_BUILD_APPS "Sets whether the apps should be build." OFF)

# Variables
set(SOURCE_ROOT Source/C++)
set(SOURCE_CODECS ${SOURCE_ROOT}/Codecs)
set(SOURCE_CORE ${SOURCE_ROOT}/Core)
set(SOURCE_CRYPTO ${SOURCE_ROOT}/Crypto)
set(SOURCE_METADATA ${SOURCE_ROOT}/MetaData)
set(SOURCE_SYSTEM ${SOURCE_ROOT}/System)

# Compiler warning and optimization flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")

if(MSVC)
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /GL")
	set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /LTCG")
	
	add_definitions(/wd4018)
	add_definitions(/wd4061)
	add_definitions(/wd4242)
	add_definitions(/wd4244)
	add_definitions(/wd4355)
	add_definitions(/wd4365)
	add_definitions(/wd4514)
	add_definitions(/wd4625)
	add_definitions(/wd4626)
	add_definitions(/wd4710)
	add_definitions(/wd4774)
	add_definitions(/wd4820)
	add_definitions(/wd5026)
	add_definitions(/wd5027)
	add_definitions(/wd5045)
endif(MSVC)

if (EMSCRIPTEN)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-warn-absolute-paths")
endif()

# AP4 Library
file(GLOB AP4_SOURCES
  ${SOURCE_CODECS}/*.cpp
  ${SOURCE_CORE}/*.cpp
  ${SOURCE_CRYPTO}/*.cpp
  ${SOURCE_METADATA}/*.cpp
  ${SOURCE_SYSTEM}/StdC/*.cpp
)

if(WIN32)
  set(AP4_SOURCES ${AP4_SOURCES} ${SOURCE_SYSTEM}/Win32/Ap4Win32Random.cpp)
else()
  set(AP4_SOURCES ${AP4_SOURCES} ${SOURCE_SYSTEM}/Posix/Ap4PosixRandom.cpp)
endif()

add_library(ap4 STATIC ${AP4_SOURCES})

# Includes
include_directories(
  ${SOURCE_CORE}
  ${SOURCE_CODECS}
  ${SOURCE_CRYPTO}
  ${SOURCE_METADATA}
)

if (BENTO4_BUILD_APPS)
	# Apps
	set(BENTO4_APPS "Aac2Mp4;Mp42Aac;Mp42Ts;Mp42Hls;Mp4Compact;Mp4DcfPackager;Mp4Decrypt;Mp4Dump;Mp4Edit;Mp4Encrypt;Mp4Extract;Mp4Fragment;Mp4Info;Mp4Mux;Mp4Split;Mp4Tag;Mp4Diff;Mp4IframeIndex")
	foreach(app ${BENTO4_APPS})
	  string(TOLOWER ${app} binary_name)
	  add_executable(${binary_name} ${SOURCE_ROOT}/Apps/${app}/${app}.cpp)
	  target_link_libraries(${binary_name} ap4)
	endforeach()
endif(BENTO4_BUILD_APPS)
