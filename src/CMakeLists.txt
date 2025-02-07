cmake_minimum_required(VERSION 3.2)
project(simple CXX)

if(SIMPLE_WITH_JIEBA)
  include(ExternalProject)
  ExternalProject_Add(
      cppjieba
      PREFIX ${CMAKE_BINARY_DIR}/cppjieba
      GIT_REPOSITORY https://github.com/yanyiwu/cppjieba.git
      CONFIGURE_COMMAND ""
      GIT_TAG 194c144d8b5ed1baf3190d07c5226e804454ab47
      BUILD_COMMAND cmake -E echo "Skipping build cppjieba."
      INSTALL_COMMAND cmake -E echo "Skipping install cppjieba."
      LOG_DOWNLOAD ON
      )
  ExternalProject_Get_Property(cppjieba source_dir)
endif()

set(SOURCE_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/pinyin.h
    ${CMAKE_CURRENT_SOURCE_DIR}/simple_highlight.h
    ${CMAKE_CURRENT_SOURCE_DIR}/simple_tokenizer.h
    ${CMAKE_CURRENT_SOURCE_DIR}/pinyin.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/simple_highlight.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/simple_tokenizer.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/entry.cc
)

OPTION(BUILD_STATIC "Option to build static lib" OFF)
if (IOS OR BUILD_STATIC)
  # iOS only support static library.
  add_library(simple STATIC ${SOURCE_FILES})
else()
  add_library(simple SHARED ${SOURCE_FILES})
endif()

if(SIMPLE_WITH_JIEBA)
  add_dependencies(simple cppjieba)
  include_directories(${SQLITE3_HEADERS_DIR} ${source_dir}/include ${source_dir}/deps)
  target_include_directories(simple INTERFACE ${SQLITE3_HEADERS_DIR} ${source_dir}/include ${source_dir}/deps)
  # for tests only
  if(MSVC)
    add_custom_command(TARGET simple PRE_BUILD
                      COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/test/dict
                      COMMAND ${CMAKE_COMMAND} -E copy_directory ${source_dir}/dict ${CMAKE_BINARY_DIR}/test/dict)
  else()
    add_custom_command(TARGET simple PRE_BUILD
                      COMMAND ${CMAKE_COMMAND} -E copy_directory
                      ${source_dir}/dict/ $<TARGET_FILE_DIR:simple>/../test/dict/)
  endif()
  INSTALL(DIRECTORY ${source_dir}/dict/ DESTINATION bin/dict FILES_MATCHING PATTERN "*.utf8")
else()
  include_directories(${SQLITE3_HEADERS_DIR})
  target_include_directories(simple INTERFACE ${SQLITE3_HEADERS_DIR})
endif()

if(BUILD_SQLITE3)
  target_link_libraries(simple PUBLIC coverage_config PRIVATE PINYIN_TEXT SQLite3)
else()
  target_link_libraries(simple PUBLIC coverage_config PRIVATE PINYIN_TEXT)
endif()

install(TARGETS simple DESTINATION bin)
