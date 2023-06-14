# Initiate with
# cmake -DCMAKE_TOOLCHAIN_FILE=../agon-sdcc.cmake ..

cmake_minimum_required(VERSION 3.20)

# Include local cmake folder for extensions
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

project(Agon-sdcc-template C)
enable_language(ASM)

set(CMAKE_C_STANDARD 21)

set_source_files_properties(crt/crt0.s PROPERTIES LANGUAGE ASM)
set_source_files_properties(lib/mos.s  PROPERTIES LANGUAGE ASM)

set(SOURCE_FILES crt/crt0.s lib/mos.s src/main.c)
set(PROJECT_NAME main)

add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_compile_options(${PROJECT_NAME} PRIVATE -mez80_z80 -Iinclude/ --reserve-regs-iy --std-c2x --fomit-frame-pointer)
target_link_options(${PROJECT_NAME} PRIVATE --no-std-crt0 -mez80_z80 --code-loc 100)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND sdldz80 -nf `basename $<TARGET_FILE:${PROJECT_NAME}> .ihx`.lk
    COMMAND sdobjcopy -I ihex -O binary $<TARGET_FILE:${PROJECT_NAME}> main.bin
    COMMAND ${CMAKE_COMMAND} -E copy main.bin ${CMAKE_SOURCE_DIR}/main.bin
    COMMENT "Running post-build steps"
)
