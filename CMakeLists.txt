
cmake_minimum_required(VERSION 3.1)
project(opt-parser)


option(OPT_PARSER_INCLUDE_TEST "include test files" OFF)

include_directories(include)
# in your file
#include_directories(fmt/include)

if (OPT_PARSER_INCLUDE_TEST)
    add_subdirectory(test)
endif(OPT_PARSER_INCLUDE_TEST)