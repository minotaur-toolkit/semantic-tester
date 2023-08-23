cmake_minimum_required(VERSION 3.8)
include(ProcessorCount)

project(SemanticTester)

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_FLAGS                "-Wall -march=native -fPIC ${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS_DEBUG          "-g -Og ${CMAKE_CXX_FLAGS_DEBUG}")
set(CMAKE_CXX_FLAGS_RELEASE        "-O3 ${CMAKE_CXX_FLAGS_RELEASE}")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELEASE} -g ${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")

find_package(LLVM REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
include(AddLLVM)

include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

find_package(Alive2)
include_directories(${ALIVE2_SOURCE_DIR})

include_directories(include)

set(INTRINSICS_TESTER_SRCS
x86Intrin.cpp
compareFunctions.cpp
)

add_library(tester STATIC ${INTRINSICS_TESTER_SRCS})
set(ALIVE_LIBS_LLVM tester ${ALIVE_LIBS_LLVM})

add_llvm_executable(intrinsics-tester
  "intrinsics-tester.cpp"
  ${INTRINSICS_TESTER_SRCS}
)


llvm_map_components_to_libnames(LLVM_LIBS
  analysis core irreader passes scalaropts support transformutils targetparser)

llvm_map_components_to_libnames(tester_libs ExecutionEngine orcjit native target jitlink)
target_link_libraries(intrinsics-tester PRIVATE ${ALIVE_LIBS_LLVM} ${llvm_libs} ${tester_libs})