cmake_minimum_required(VERSION 3.20)
project(cmake_fetch_content_example LANGUAGES CXX)

# option() honor variables
if (POLICY CMP0077)
  cmake_policy(SET CMP0077 NEW)
endif (POLICY CMP0077)

include(FetchContent)

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
#   Attempts to add an alias for a target if the target does not exist.
function(maybe_add_alias my_target my_alias)
  if(NOT TARGET ${my_alias})
    if(TARGET ${my_target})
      add_library(${my_target} ALIAS ${my_alias})
    endif(TARGET ${my_target})
  endif(NOT TARGET ${my_alias})
endfunction()


# check_absl_target(target)
#   Attempts to add an alias for an absl namespace target
#   before running check_target(target)
function(check_absl_target my_target)
  string(FIND ${my_target} "::" _has_namespace)
  if(${_has_namespace})
    string(REPLACE "::" "_" _my_alias ${my_target})
    maybe_add_alias(${my_target} ${_my_alias})
  endif(${_has_namespace})

  check_target("${my_target}")
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
find_package(GTest REQUIRED)

check_target(GTest::gtest)
check_target(GTest::gtest_main)
check_target(GTest::gmock)
check_target(GTest::gmock_main)


# Include nlohmann_json using FetchContent

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

add_library(nlohmann_json::nlohmann_json ALIAS nlohmann_json)



# Include Abseil using FetchContent.

set(ABSL_USE_EXTERNAL_GOOGLETEST ON CACHE INTERNAL "" FORCE)
set(ABSL_BUILD_TESTING OFF CACHE INTERNAL "" FORCE)
set(ABSL_PROPAGATE_CXX_STD ON CACHE INTERNAL "" FORCE)

FetchContent_Declare(
  com_google_absl
  URL        https://github.com/abseil/abseil-cpp/archive/20211102.0.tar.gz
  URL_HASH   SHA256=dcf71b9cba8dc0ca9940c4b316a0c796be8fab42b070bb6b7cab62b48f0e66c4)


FetchContent_MakeAvailable(com_google_absl)






check_absl_target(absl::algorithm)
check_absl_target(absl::algorithm_container)
check_absl_target(absl::base)
check_absl_target(absl::core_headers)
check_absl_target(absl::log_severity)
check_absl_target(absl::dynamic_annotations)
check_absl_target(absl::cleanup)
check_absl_target(absl::btree)
check_absl_target(absl::fixed_array)
check_absl_target(absl::flat_hash_map)
check_absl_target(absl::flat_hash_set)
check_absl_target(absl::inlined_vector)
check_absl_target(absl::node_hash_map)
check_absl_target(absl::node_hash_set)
check_absl_target(absl::debugging)
check_absl_target(absl::failure_signal_handler)
check_absl_target(absl::leak_check)
check_absl_target(absl::leak_check_disable)
check_absl_target(absl::stacktrace)
check_absl_target(absl::symbolize)
check_absl_target(absl::flags)
check_absl_target(absl::flags_commandlineflag)
check_absl_target(absl::flags_config)
check_absl_target(absl::flags_marshalling)
check_absl_target(absl::flags_parse)
check_absl_target(absl::flags_reflection)
check_absl_target(absl::flags_usage)
check_absl_target(absl::bind_front)
check_absl_target(absl::function_ref)
check_absl_target(absl::hash)
check_absl_target(absl::hash_testing)
check_absl_target(absl::memory)
check_absl_target(absl::meta)
check_absl_target(absl::type_traits)
check_absl_target(absl::bits)
check_absl_target(absl::int128)
check_absl_target(absl::numeric)
check_absl_target(absl::numeric_representation)
check_absl_target(absl::exponential_biased)
check_absl_target(absl::periodic_sampler)
check_absl_target(absl::sample_recorder)
check_absl_target(absl::random_bit_gen_ref)
check_absl_target(absl::random_distributions)
check_absl_target(absl::random_mocking_bit_gen)
check_absl_target(absl::random_random)
check_absl_target(absl::random_seed_gen_exception)
check_absl_target(absl::random_seed_sequences)
check_absl_target(absl::status)
check_absl_target(absl::statusor)
check_absl_target(absl::cord)
check_absl_target(absl::cord_test_helpers)
check_absl_target(absl::str_format)
check_absl_target(absl::strings)
check_absl_target(absl::synchronization)
check_absl_target(absl::civil_time)
check_absl_target(absl::time)
check_absl_target(absl::time_zone)
check_absl_target(absl::any)
check_absl_target(absl::bad_any_cast)
check_absl_target(absl::bad_optional_access)
check_absl_target(absl::bad_variant_access)
check_absl_target(absl::compare)
check_absl_target(absl::optional)
check_absl_target(absl::span)
check_absl_target(absl::variant)
check_absl_target(absl::utility)
check_absl_target(absl::base)
check_absl_target(absl::base)


# dump_cmake_variables()
# dump_cmake_targets(${CMAKE_BINARY_DIR})

add_subdirectory(example)

