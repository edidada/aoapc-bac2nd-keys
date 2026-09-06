# 《算法竞赛入门经典（第 2 版）》习题代码构建脚本
#
# 仓库中每个 .cc / .cpp 都是一个独立的完整程序（含 main），
# 因此这里为每一个源文件生成一个可执行目标，目标名由所在目录 + 文件名组成，
# 例如：ch03_UVa1585、03_regional_LA2237、contrib_zhangminchen_ch07_UVA12558。
#
# 用法：
#   cmake -S . -B build
#   cmake --build build                 # 编译全部
#   cmake --build build --target ch03_UVa1585   # 只编译某一题

cmake_minimum_required(VERSION 3.16)
project(aoapc-bac2nd-keys LANGUAGES CXX)

# 这些题解绝大多数编写于 2014~2018 年，使用 C++11 足够；
# 保留 GNU 扩展(gnu++11)，避免像 gets() 这类传统接口无法编译。
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

# 统一把可执行文件输出到 build/bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# 是否一并构建 contrib/ 下贡献者的代码
option(INCLUDE_CONTRIB "Build problem solutions under contrib/" ON)

set(SOURCE_DIRS
    ch03 ch04 ch05 ch06 ch07 ch08 ch09 ch10 ch11 ch12 03_regional)
if(INCLUDE_CONTRIB)
  list(APPEND SOURCE_DIRS contrib)
endif()

set(ALL_PROBLEM_SOURCES "")
foreach(dir IN LISTS SOURCE_DIRS)
  file(GLOB_RECURSE dir_sources
       CONFIGURE_DEPENDS
       "${CMAKE_CURRENT_SOURCE_DIR}/${dir}/*.cc"
       "${CMAKE_CURRENT_SOURCE_DIR}/${dir}/*.cpp")
  list(APPEND ALL_PROBLEM_SOURCES ${dir_sources})
endforeach()

if(NOT ALL_PROBLEM_SOURCES)
  message(FATAL_ERROR "No source files found, please check the source directories.")
endif()

list(SORT ALL_PROBLEM_SOURCES)

foreach(src IN LISTS ALL_PROBLEM_SOURCES)
  # 目标名形如 <所在目录路径>/<文件名>，目录分隔符替换为下划线，保证全局唯一
  file(RELATIVE_PATH rel "${CMAKE_CURRENT_SOURCE_DIR}" "${src}")
  get_filename_component(rel_dir "${rel}" DIRECTORY)
  get_filename_component(src_name "${src}" NAME_WE)
  string(REPLACE "/" "_" dir_token "${rel_dir}")
  set(target_name "${dir_token}_${src_name}")
  add_executable(${target_name} "${src}")
  set_target_properties(${target_name} PROPERTIES OUTPUT_NAME "${target_name}")
endforeach()

list(LENGTH ALL_PROBLEM_SOURCES total_sources)
message(STATUS "Total problem solutions to build: ${total_sources}")
