# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 3.8.2)

if(GRAPHQL_UPDATE_SAMPLES)
  # unifiedschema
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/unified)

  add_custom_command(
    OUTPUT
      ${CMAKE_CURRENT_SOURCE_DIR}/unified/TodaySchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/unified/TodaySchema.h
    COMMAND schemagen --schema="${CMAKE_CURRENT_SOURCE_DIR}/schema.today.graphql" --prefix="Today" --namespace="today"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      TodaySchema.cpp
      TodaySchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/unified
    DEPENDS schemagen graphqlpeg schema.today.graphql
    WORKING_DIRECTORY unified
    COMMENT "Generating mock TodaySchema files")

  # unifiedschema_nointrospection
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/unified_nointrospection)

  add_custom_command(
    OUTPUT
      ${CMAKE_CURRENT_SOURCE_DIR}/unified_nointrospection/TodaySchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/unified_nointrospection/TodaySchema.h
    COMMAND schemagen --schema="${CMAKE_CURRENT_SOURCE_DIR}/schema.today.graphql" --prefix="Today" --namespace="today" --no-introspection
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      TodaySchema.cpp
      TodaySchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/unified_nointrospection
    DEPENDS schemagen graphqlpeg schema.today.graphql
    WORKING_DIRECTORY unified_nointrospection
    COMMENT "Generating mock TodaySchema files without Introspection (--no-introspection)")

  # validationschema
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/validation)

  add_custom_command(
    OUTPUT
      ${CMAKE_CURRENT_SOURCE_DIR}/validation/ValidationSchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/validation/ValidationSchema.h
    COMMAND schemagen --schema="${CMAKE_CURRENT_SOURCE_DIR}/schema.validation.graphql" --prefix="Validation" --namespace="validation"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      ValidationSchema.cpp
      ValidationSchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/validation
    DEPENDS schemagen graphqlpeg schema.validation.graphql
    WORKING_DIRECTORY validation
    COMMENT "Generating ValidationSchema files")

  # separateschema
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/separate)

  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/separate/today_schema_files
    COMMAND schemagen --schema="${CMAKE_CURRENT_SOURCE_DIR}/schema.today.graphql" --prefix="Today" --namespace="today" --separate-files > today_schema_files
    COMMAND ${CMAKE_COMMAND} -E copy_if_different today_schema_files ${CMAKE_CURRENT_SOURCE_DIR}/separate
    DEPENDS schemagen graphqlpeg schema.today.graphql
    WORKING_DIRECTORY separate
    COMMENT "Generating mock TodaySchema (--separate-files)")

  # separateschema_nointrospection
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/separate_nointrospection)

  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection/today_schema_files
    COMMAND schemagen --schema="${CMAKE_CURRENT_SOURCE_DIR}/schema.today.graphql" --prefix="Today" --namespace="today" --no-introspection --separate-files > today_schema_files
    COMMAND ${CMAKE_COMMAND} -E copy_if_different today_schema_files ${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection
    DEPENDS schemagen graphqlpeg schema.today.graphql
    WORKING_DIRECTORY separate_nointrospection
    COMMENT "Generating mock TodaySchema without Introspection (--no-introspection --separate-files)")

  add_custom_command(
    OUTPUT updated_samples
    COMMAND ${CMAKE_COMMAND}
      "-DSCHEMA_SOURCE_LIST=today_schema_files"
      "-DSCHEMA_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/separate"
      "-DSCHEMA_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/separate"
      -P "${CMAKE_CURRENT_SOURCE_DIR}/update_samples.cmake"
    COMMAND ${CMAKE_COMMAND}
      "-DSCHEMA_SOURCE_LIST=today_schema_files"
      "-DSCHEMA_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection"
      "-DSCHEMA_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/separate_nointrospection"
      -P "${CMAKE_CURRENT_SOURCE_DIR}/update_samples.cmake"
    COMMAND ${CMAKE_COMMAND} -E touch updated_samples
    DEPENDS 
      ${CMAKE_CURRENT_SOURCE_DIR}/unified/TodaySchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/unified/TodaySchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/unified_nointrospection/TodaySchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/unified_nointrospection/TodaySchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/validation/ValidationSchema.cpp
      ${CMAKE_CURRENT_SOURCE_DIR}/validation/ValidationSchema.h
      ${CMAKE_CURRENT_SOURCE_DIR}/separate/today_schema_files
      ${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection/today_schema_files
    COMMENT "Updating sample files")

  add_custom_target(update_samples ALL
    DEPENDS updated_samples)
endif()

# separateschema
set(SEPARATE_SCHEMA_PATHS "")
file(STRINGS ${CMAKE_CURRENT_SOURCE_DIR}/separate/today_schema_files SEPARATE_SCHEMA_FILES)
foreach(CPP_FILE IN LISTS SEPARATE_SCHEMA_FILES)
  list(APPEND SEPARATE_SCHEMA_PATHS "${CMAKE_CURRENT_SOURCE_DIR}/separate/${CPP_FILE}")
endforeach(CPP_FILE)

add_library(separateschema STATIC ${SEPARATE_SCHEMA_PATHS})
target_link_libraries(separateschema PUBLIC graphqlintrospection)
target_compile_definitions(separateschema PUBLIC IMPL_SEPARATE_TODAY)
target_include_directories(separateschema PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include
  separate)

if(GRAPHQL_UPDATE_SAMPLES)
  # wait for the sample update to complete
  add_dependencies(separateschema update_samples)
endif()

# separateschema_nointrospection
set(SEPARATE_NOINTROSPECTION_SCHEMA_PATHS "")
file(STRINGS ${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection/today_schema_files SEPARATE_NOINTROSPECTION_SCHEMA_FILES)
foreach(CPP_FILE IN LISTS SEPARATE_NOINTROSPECTION_SCHEMA_FILES)
  list(APPEND SEPARATE_NOINTROSPECTION_SCHEMA_PATHS "${CMAKE_CURRENT_SOURCE_DIR}/separate_nointrospection/${CPP_FILE}")
endforeach(CPP_FILE)

add_library(separateschema_nointrospection STATIC ${SEPARATE_NOINTROSPECTION_SCHEMA_PATHS})
target_link_libraries(separateschema_nointrospection PUBLIC graphqlservice)
target_compile_definitions(separateschema_nointrospection PUBLIC IMPL_SEPARATE_TODAY)
target_include_directories(separateschema_nointrospection PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include
  separate_nointrospection)

if(GRAPHQL_UPDATE_SAMPLES)
  # wait for the sample update to complete
  add_dependencies(separateschema_nointrospection update_samples)
endif()

# separategraphql
add_library(separategraphql STATIC today/TodayMock.cpp)
target_link_libraries(separategraphql PUBLIC separateschema)
target_include_directories(separategraphql PUBLIC today)

# separategraphql_nointrospection
add_library(separategraphql_nointrospection STATIC today/TodayMock.cpp)
target_link_libraries(separategraphql_nointrospection PUBLIC separateschema_nointrospection)
target_include_directories(separategraphql_nointrospection PUBLIC today)

# sample
add_executable(sample today/sample.cpp)
target_link_libraries(sample PRIVATE
  separategraphql
  graphqljson)
target_include_directories(sample PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include)

# sample_nointrospection
add_executable(sample_nointrospection today/sample.cpp)
target_link_libraries(sample_nointrospection PRIVATE
  separategraphql_nointrospection
  graphqljson)
target_include_directories(sample_nointrospection PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include)

if(WIN32 AND BUILD_SHARED_LIBS)
  add_custom_command(OUTPUT copied_sample_dlls
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      $<TARGET_FILE:graphqlservice>
      $<TARGET_FILE:graphqlintrospection>
      $<TARGET_FILE:graphqljson>
      $<TARGET_FILE:graphqlpeg>
      $<TARGET_FILE:graphqlresponse>
      ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E touch copied_sample_dlls
    DEPENDS
      graphqlservice
      graphqlintrospection
      graphqljson
      graphqlpeg
      graphqlresponse)

  add_custom_target(copy_sample_dlls DEPENDS copied_sample_dlls)

  add_dependencies(sample copy_sample_dlls)
  add_dependencies(sample_nointrospection copy_sample_dlls)
endif()

# benchmark
add_executable(benchmark today/benchmark.cpp)
target_link_libraries(benchmark PRIVATE
  separategraphql
  graphqljson)
target_include_directories(benchmark PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include)

# benchmark_nointrospection
add_executable(benchmark_nointrospection today/benchmark.cpp)
target_link_libraries(benchmark_nointrospection PRIVATE
  separategraphql_nointrospection
  graphqljson)
target_include_directories(benchmark_nointrospection PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include)

if(WIN32 AND BUILD_SHARED_LIBS)
  add_dependencies(benchmark copy_sample_dlls)
  add_dependencies(benchmark_nointrospection copy_sample_dlls)
endif()

if(GRAPHQL_BUILD_TESTS)
  # tests
  add_library(unifiedschema STATIC unified/TodaySchema.cpp)
  target_link_libraries(unifiedschema PUBLIC graphqlintrospection)
  target_include_directories(unifiedschema PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/../include
    ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include
    unified)

  if(GRAPHQL_UPDATE_SAMPLES)
    # wait for the sample update to complete
    add_dependencies(unifiedschema update_samples)
  endif()

  add_library(unifiedgraphql STATIC today/TodayMock.cpp)
  target_link_libraries(unifiedgraphql PUBLIC unifiedschema)
  target_include_directories(unifiedgraphql PUBLIC today)
  add_bigobj_flag(unifiedgraphql)

  add_library(unifiedschema_nointrospection STATIC unified_nointrospection/TodaySchema.cpp)
  target_link_libraries(unifiedschema_nointrospection PUBLIC graphqlservice)
  target_include_directories(unifiedschema_nointrospection PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/../include
    ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include
    unified_nointrospection)

  if(GRAPHQL_UPDATE_SAMPLES)
    # wait for the sample update to complete
    add_dependencies(unifiedschema_nointrospection update_samples)
  endif()

  add_library(unifiedgraphql_nointrospection STATIC today/TodayMock.cpp)
  target_link_libraries(unifiedgraphql_nointrospection PUBLIC unifiedschema_nointrospection)
  target_include_directories(unifiedgraphql_nointrospection PUBLIC today)
  add_bigobj_flag(unifiedgraphql_nointrospection)

  add_library(validationgraphql STATIC
    validation/ValidationMock.cpp
    validation/ValidationSchema.cpp)
  target_link_libraries(validationgraphql PUBLIC graphqlintrospection)
  target_include_directories(validationgraphql PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/../include
    ${CMAKE_CURRENT_SOURCE_DIR}/../PEGTL/include
    validation)
  add_bigobj_flag(validationgraphql)

  if(GRAPHQL_UPDATE_SAMPLES)
    # wait for the sample update to complete
    add_dependencies(validationgraphql update_samples)
  endif()
endif()
