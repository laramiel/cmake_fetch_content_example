cmake_minimum_required(VERSION 3.20)
project(cmake_fetch_content_example LANGUAGES CXX)

####################
# SET POLICY       #
####################

# Compiler id for Apple Clang is now AppleClang.
if (POLICY CMP0025)
  cmake_policy(SET CMP0025 NEW)
endif (POLICY CMP0025)

# Project version variables are the empty string if version is unspecified
if (POLICY CMP0048)
  cmake_policy(SET CMP0048 NEW)
endif (POLICY CMP0048)

# if command can use IN_LIST
if (POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)
endif (POLICY CMP0057)

# if command can use TEST
if (POLICY CMP0064)
  cmake_policy(SET CMP0064 NEW)
endif (POLICY CMP0064)

# option() honor variables
if (POLICY CMP0077)
  cmake_policy(SET CMP0077 NEW)
endif (POLICY CMP0077)

# Allow the user to specify the MSVC runtime
if (POLICY CMP0091)
  cmake_policy(SET CMP0091 NEW)
endif (POLICY CMP0091)

####################
# HELPER FUNCTIONS #
####################

# check_target(target)
#   Errors if the targetg does not exist.
function(check_target my_target)
  if(NOT TARGET ${my_target})
    message(FATAL_ERROR " Building requires a ${my_target} CMake target.")
  endif(NOT TARGET ${my_target})
endfunction()

# maybe_add_alias(target alias)
#   Attempts to add an alias for a target if the alias does not exist.
function(maybe_add_alias my_target my_alias)
  if(NOT TARGET ${my_alias})
    if(TARGET ${my_target})
      add_library(${my_target} ALIAS ${my_alias})
    endif(TARGET ${my_target})
  endif(NOT TARGET ${my_alias})
endfunction()


# dump_cmake_variables()
#   Dumps all the CMAKE variables.
function(dump_cmake_variables)
  # https://stackoverflow.com/questions/9298278/cmake-print-out-all-accessible-variables-in-a-script
  get_cmake_property(_variableNames VARIABLES)
  list (SORT _variableNames)
  foreach (_variableName ${_variableNames})
    if (ARGV0)
      unset(MATCHED)
      string(REGEX MATCH ${ARGV0} MATCHED ${_variableName})
      if (NOT MATCHED)
         continue()
      endif()
    endif()
    message(STATUS "${_variableName}=${${_variableName}}")
  endforeach()
endfunction()

# dump_cmake_targets( <DIRECTORY> )
#   Dumps all the CMAKE targets under the <DIRECTORY>.
function(dump_cmake_targets directory)
  get_property(imported_targets DIRECTORY ${directory} PROPERTY IMPORTED_TARGETS)
  foreach(_target ${imported_targets})
    message(STATUS "+ ${_target}")
  endforeach()

  get_property(dir_targets DIRECTORY ${directory} PROPERTY BUILDSYSTEM_TARGETS)
  foreach(_target ${dir_targets})
    get_target_property(_type ${_target} TYPE)
    message(STATUS "+ ${_target}  ${_type}")
  endforeach()

  get_property(sub_directories DIRECTORY ${directory} PROPERTY SUBDIRECTORIES)
  foreach(directory ${sub_directories})
    dump_cmake_targets(${directory})
  endforeach()
endfunction()

####################
# IMPLEMENTATION   #
####################

# Set all outputs to a single /bin directory.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_CXX_STANDARD 17)

## Abseil requires PIC code; we generally use the same.
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

find_package(Threads REQUIRED)

include(CTest)
include(CMakePackageConfigHelpers)
include(FetchContent)

########
# Install GTest using FetchContent.

FetchContent_Declare(
  com_google_googletest
  URL        https://github.com/google/googletest/archive/3e0e32ba300ce8afe695ad3ba7e81b21b7cf237a.zip
  URL_HASH   SHA256=84bf0acb4a7ed172ffdd806bb3bef76ad705f4ea63ac7175cd7c86df2a017d17)

FetchContent_MakeAvailable(com_google_googletest)

check_target(GTest::gtest)
check_target(GTest::gtest_main)
check_target(GTest::gmock)
check_target(GTest::gmock_main)


########
# Install Abseil using FetchContent.

set(ABSL_PROPAGATE_CXX_STD ON CACHE INTERNAL "" FORCE)
set(ABSL_BUILD_TESTING OFF CACHE INTERNAL "" FORCE)
set(ABSL_USE_EXTERNAL_GOOGLETEST ON CACHE INTERNAL "" FORCE)
set(ABSL_FIND_GOOGLETEST OFF CACHE INTERNAL "" FORCE)

FetchContent_Declare(
  com_google_absl
  URL        https://github.com/abseil/abseil-cpp/archive/20211102.0.tar.gz
  URL_HASH   SHA256=dcf71b9cba8dc0ca9940c4b316a0c796be8fab42b070bb6b7cab62b48f0e66c4)

FetchContent_MakeAvailable(com_google_absl)

# The Abseil library aliases are not installed via FetchContent

maybe_add_alias(absl_strings absl::strings)


########
# Install nlohmann_json using FetchContent

FetchContent_Declare(
  com_github_nlohmann_json
  URL        https://github.com/nlohmann/json/releases/download/v3.10.2/include.zip
  URL_HASH   SHA256=61e605be15e88deeac4582aaf01c09d616f8302edde7adcaba9261ddc3b4ceca)


# nlohmann_json install doesn't appear to work with FetchContent. :/

FetchContent_GetProperties(com_github_nlohmann_json)
if(NOT com_github_nlohmann_json_POPULATED)
  FetchContent_Populate(com_github_nlohmann_json)
endif()

add_library(nlohmann_json INTERFACE)
target_include_directories(nlohmann_json INTERFACE
      "${com_github_nlohmann_json_SOURCE_DIR}/single_include")

maybe_add_alias(nlohmann_json nlohmann_json::nlohmann_json)


########

# dump_cmake_variables()
# dump_cmake_targets(${CMAKE_BINARY_DIR})

add_subdirectory(example)

