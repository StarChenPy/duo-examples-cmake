cmake_minimum_required(VERSION 3.16)

project(duo-examples-cmake)

# 检查并下载 SDK
find_program(WGET_PROGRAM wget)
if(NOT WGET_PROGRAM)
    message(FATAL_ERROR "wget not found")
endif()

set(SDK_URL "https://github.com/milkv-duo/duo-app-sdk/releases/download/duo-app-sdk-v1.2.0/duo-sdk-v1.2.0.tar.gz")
set(SDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/buildtools")
set(SDK_FILE "${SDK_DIR}/duo-sdk-v1.2.0.tar.gz")
set(MILKV_DUO_SDK "${SDK_DIR}/duo-sdk")
set(TOOLCHAIN_DIR "${MILKV_DUO_SDK}/riscv64-linux-musl-x86_64")
set(TOOLCHAIN_PREFIX "${TOOLCHAIN_DIR}/bin/riscv64-unknown-linux-musl-")

if(NOT EXISTS "${MILKV_DUO_SDK}")
    file(DOWNLOAD ${SDK_URL} ${SDK_FILE})
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xzf ${SDK_FILE} WORKING_DIRECTORY ${SDK_DIR})
endif()

# 设置环境变量
set(ENV{TOOLCHAIN_PREFIX} ${TOOLCHAIN_PREFIX})
set(ENV{SYSROOT} "${MILKV_DUO_SDK}/rootfs")
set(ENV{CC} "${TOOLCHAIN_PREFIX}gcc")
set(ENV{CFLAGS} "-mcpu=c906fdv -march=rv64imafdcv0p7xthead -mcmodel=medany -mabi=lp64d -Os")
set(ENV{LDFLAGS} "-D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64")

# 设置编译器
set(CMAKE_C_COMPILER $ENV{TOOLCHAIN_PREFIX}gcc)

# 添加编译器标志
set(CMAKE_C_FLAGS "$ENV{CFLAGS} -I$ENV{SYSROOT}/usr/include")

# 添加链接器标志
set(CMAKE_EXE_LINKER_FLAGS "$ENV{LDFLAGS} -L$ENV{SYSROOT}/lib -L$ENV{SYSROOT}/usr/lib -lwiringx")

# 添加源文件
file(GLOB SOURCES "src/*.c")

# 添加文件目标
add_executable(${PROJECT_NAME} ${SOURCES})
include_directories(include)

# 链接库文件
target_link_libraries(${PROJECT_NAME} wiringx)
