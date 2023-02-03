set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.24)

set(CMAKE_C_COMPILER /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-gcc)
set(CMAKE_CXX_COMPILER /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-g++)
set(CMAKE_ASM_COMPILER /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-gcc)
set(CMAKE_AR /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-ar)
set(CMAKE_OBJCOPY /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-objcopy)
set(CMAKE_OBJDUMP /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-objdump)
set(SIZE /opt/xpack/riscv-none-elf-gcc/bin/riscv-none-elf-size)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# project settings
project(CH32V003_cmake C CXX ASM)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 11)

#add_compile_options(-mfloat-abi=soft)
add_compile_options(-march=rv32ec_zicsr -mabi=ilp32e)
add_compile_options(-ffunction-sections -fdata-sections -fno-common -fmessage-length=0 -msmall-data-limit=0 -msave-restore -fsigned-char)

# uncomment to mitigate c++17 absolute addresses warnings
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-register")

# Enable assembler files preprocessing
add_compile_options($<$<COMPILE_LANGUAGE:ASM>:-x$<SEMICOLON>assembler-with-cpp>)

if ("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    message(STATUS "Maximum optimization for speed")
    add_compile_options(-Ofast)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo")
    message(STATUS "Maximum optimization for speed, debug info included")
    add_compile_options(-Ofast -g)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "MinSizeRel")
    message(STATUS "Maximum optimization for size")
    add_compile_options(-Os)
else ()
    message(STATUS "Minimal optimization, debug info included")
    add_compile_options(-Og -g)
endif ()

include_directories(User/inc Peripheral/inc Debug Core)

#add_definitions(-DDEBUG -DUSE_HAL_DRIVER -DSTM32G474xx)

file(GLOB_RECURSE SOURCES "Core/*.*" "Debug/*.*" "User/*.*" "Peripheral/*.*" "Startup/*.*")

set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/Ld/Link.ld)

add_link_options(-march=rv32ec -mabi=ilp32e)
#add_link_options(-ffunction-sections -fdata-sections -fno-common -fmessage-length=0 -msmall-data-limit=0 -msave-restore -fsigned-char)
add_link_options(-msmall-data-limit=0 -msave-restore -fmessage-length=0 -fsigned-char -ffunction-sections -fdata-sections -fno-common)
add_link_options(-T ${LINKER_SCRIPT})
add_link_options(-nostartfiles -Xlinker --gc-sections)
add_link_options(--specs=nano.specs --specs=nosys.specs)
add_link_options(-Wl,--print-memory-usage,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map)

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)
set(LST_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.lst)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJDUMP} --all-headers --demangle --disassemble $<TARGET_FILE:${PROJECT_NAME}.elf> > ${LST_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMAND ${SIZE} $<TARGET_FILE:${PROJECT_NAME}.elf>
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")
