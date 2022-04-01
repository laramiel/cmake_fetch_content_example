
add_executable(example)

target_sources(example PRIVATE example.cc)

target_link_libraries(example PUBLIC absl::strings)
