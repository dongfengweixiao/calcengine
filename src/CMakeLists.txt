# The Flutter tooling requires that developers have CMake 3.10 or later
# installed. You should not increase this version, as doing so will cause
# the plugin to fail to compile for some customers of the plugin.
cmake_minimum_required(VERSION 3.10)

project(calcengine_library VERSION 0.0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(PCH_FILE ${CMAKE_CURRENT_SOURCE_DIR}/calculator/src/CalcManager/pch.h)
if(EXISTS ${PCH_FILE} AND NOT WIN32)
    file(READ ${PCH_FILE} PCH_CONTENT)
    
    string(REPLACE "#include <intsafe.h>" "// #include <intsafe.h>" PCH_CONTENT "${PCH_CONTENT}")
    string(REPLACE "#include <winerror.h>" "// #include <winerror.h>" PCH_CONTENT "${PCH_CONTENT}")
    
    file(WRITE ${PCH_FILE} "${PCH_CONTENT}")
    message(STATUS "Modified pch.h to comment out Windows-specific includes")
endif()

file(GLOB_RECURSE SOURCES 
  "calculator/src/CalcManager/CEngine/*.cpp" 
  "calculator/src/CalcManager/Ratpack/*.cpp"
  "calculator/src/CalcManager/*.cpp"
)
file(GLOB_RECURSE HEADERS
  "calculator/src/CalcManager/Header Files/*.h"
  "calculator/src/CalcManager/Ratpack/*.h"
  "calculator/src/CalcManager/*.h"
)
list(FILTER SOURCES EXCLUDE REGEX ".*UnitTests.*")

add_library(calcengine SHARED ${SOURCES})

target_compile_definitions(calcengine PUBLIC DART_SHARED_LIB)

# 添加包含目录
target_include_directories(calcengine PRIVATE
    "calculator/src/CalcManager"
    "calculator/src/CalcManager/Header Files"
    "calculator/src/CalcManager/Ratpack"
    "calculator/src/CalcManager/CEngine"
)

# 预编译头
target_precompile_headers(calcengine PRIVATE
  "calculator/src/CalcManager/pch.h"
)

if (ANDROID)
  # Support Android 15 16k page size
  target_link_options(calcengine PRIVATE "-Wl,-z,max-page-size=16384")
endif()
