cmake_minimum_required(VERSION 3.9) # CMP0069 NEW

project(simdjson
  DESCRIPTION "Parsing gigabytes of JSON per second"
  LANGUAGES CXX C
)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 4)
set(PROJECT_VERSION_PATCH 0)
set(SIMDJSON_LIB_VERSION "0.4.0" CACHE STRING "simdjson library version")
set(SIMDJSON_LIB_SOVERSION "2" CACHE STRING "simdjson library soversion")
set(SIMDJSON_GITHUB_REPOSITORY https://github.com/simdjson/simdjson)

include(GNUInstallDirs)
include(cmake/simdjson-flags.cmake)
include(cmake/simdjson-user-cmakecache.cmake)

if(SIMDJSON_JUST_LIBRARY)
  MESSAGE( STATUS "Building just the library, omitting all tests, tools and benchmarks." )
endif()

#
# Set up test data
#
if(NOT(SIMDJSON_JUST_LIBRARY))
  enable_testing()
  add_subdirectory(jsonchecker)
  add_subdirectory(jsonexamples)
  add_library(test-data INTERFACE)
  target_link_libraries(test-data INTERFACE jsonchecker-data jsonexamples-data)
endif()

#
# Create the top level simdjson library (must be done at this level to use both src/ and include/
# directories) and tools
#
add_subdirectory(include)
add_subdirectory(src)
add_subdirectory(windows)
if(NOT(SIMDJSON_JUST_LIBRARY))
  add_subdirectory(tools)
  add_subdirectory(singleheader)
endif()

#
# Compile tools / tests / benchmarks
#
if(NOT(SIMDJSON_JUST_LIBRARY))
  add_subdirectory(dependencies)
  add_subdirectory(tests)
  add_subdirectory(examples)
  add_subdirectory(benchmark)
  add_subdirectory(fuzz)
endif()

#
# Source files should be just ASCII
#
find_program(FIND find)
find_program(FILE file)
find_program(GREP grep)
if((FIND) AND (FILE) AND (GREP))
    add_test(
      NAME "just_ascii"
      COMMAND sh -c "${FIND}  include src windows tools singleheader tests examples benchmark -path benchmark/checkperf-reference -prune -name '*.h'  -o -name '*.cpp' -type f  -exec ${FILE} '{}' \; |${GREP} -v ASCII || exit 0  && exit 1"
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
endif()


#
# CPack
#
set(CPACK_PACKAGE_VENDOR "Daniel Lemire")
set(CPACK_PACKAGE_CONTACT "lemire@gmail.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Parsing gigabytes of JSON per second")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

set(CPACK_RPM_PACKAGE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

include(CPack)
