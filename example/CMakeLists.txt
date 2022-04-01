
add_executable(example)

target_sources(example example.cc)

target_link_libraries(example absl::strings)
