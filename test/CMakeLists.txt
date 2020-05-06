
cmake_minimum_required(VERSION 3.1)
project(opt-parser)
function(add_opt_parser_gtest test_name)
    add_executable(${test_name} ${test_name}.cpp test-main.cpp ${ARGN})
    target_link_libraries(${test_name} gtest fmt)
    target_include_directories(${test_name} SYSTEM PUBLIC gtest gmock)
    if (MINGW)
        target_link_libraries(${test_name} -static-libgcc -static-libstdc++)
    endif ()
    add_test(NAME ${test_name} COMMAND ${test_name})
endfunction()

add_opt_parser_gtest(opt_test)

