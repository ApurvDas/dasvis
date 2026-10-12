cmake_minimum_required(VERSION 3.21)

# Static vcpkg libraries + static CRT, as in WreckBox.
set(VCPKG_TARGET_TRIPLET x64-windows-static CACHE STRING "")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

project(dasvis VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_compile_options(/W4 /utf-8 /permissive-)
add_compile_definitions(UNICODE _UNICODE NOMINMAX WIN32_LEAN_AND_MEAN _CRT_SECURE_NO_WARNINGS)

find_package(nlohmann_json CONFIG REQUIRED)
find_path(DRLIBS_INCLUDE_DIR dr_wav.h REQUIRED)

# Audio decoding for the song map (WAV / MP3 / FLAC by dr_libs, everything else by Media Foundation).
add_library(dasvis_decode STATIC src/engine/decode.cpp)
target_include_directories(dasvis_decode PUBLIC src PRIVATE ${DRLIBS_INCLUDE_DIR})
target_link_libraries(dasvis_decode PUBLIC nlohmann_json::nlohmann_json PRIVATE mfplat mfreadwrite mfuuid)

# DasVis: Direct3D 11 at feature level 10_0, shaders compiled at build time by the Windows SDK's fxc.
include(cmake/dasvis.cmake)
add_library(dasvis STATIC
  src/dasvis/features.cpp
  src/dasvis/control.cpp
  src/dasvis/dasvis.cpp
  src/dasvis/songmap.cpp
  src/dasvis/palette.cpp
  src/dasvis/director.cpp
  src/dasvis/video.cpp
  src/dasvis/spout.cpp
  src/dasvis/lyrics.cpp
  ${DASVIS_GENERATED})
target_include_directories(dasvis PUBLIC src PRIVATE ${CMAKE_BINARY_DIR})
target_link_libraries(dasvis PUBLIC dasvis_decode d3d11 dxgi PRIVATE mfplat mfreadwrite mfuuid)

enable_testing()
add_executable(dasvis_tests tests/dasvis_tests.cpp)
target_link_libraries(dasvis_tests PRIVATE dasvis psapi mfplat mfreadwrite mfuuid)
target_compile_definitions(dasvis_tests PRIVATE WB_FIXTURES="${CMAKE_CURRENT_SOURCE_DIR}/tests/fixtures")
add_test(NAME dasvis_tests COMMAND dasvis_tests)
