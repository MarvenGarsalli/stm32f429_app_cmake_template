cmake_minimum_required(VERSION 3.29)

project(STM32F429I_DISCO)

set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/output)

set(DEVICE_FAMILY "STM32F429xx")
set(HSE_VALUE 8000000)

# additional compiler options: use size-optimized version of library in release build, use -O0 in debug build
if(CMAKE_BUILD_TYPE MATCHES Debug)
    set(flags -O0 -g3)
else()
    set(flags -Os -g0)
endif()

# Include toolchain file
if(USING_TEMPLATE)
    message(NOTICE "/!\ BUILD Using Template: Manually configured using toolchain file from STM32F4Template Repo!")
    message(CHECK_START "Use cmake/GNU-ARM-Toolchain.cmake")
    include("cmake/GNU-ARM-Toolchain.cmake")
    message(CHECK_PASS)
else()
    include("cmake/gcc-arm-none-eabi.cmake")
endif ()

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

include_directories(Drivers/STM32F4xx_HAL_Driver/Inc)
include_directories(Drivers/STM32F4xx_HAL_Driver/Inc/Legacy)
include_directories(Drivers/CMSIS/Device/ST/STM32F4xx/Include)
include_directories(Drivers/CMSIS/Include)

add_subdirectory(Drivers)

# Used linker file (just for testing)
get_filename_component(LINKER_SCRIPT STM32F429ZITx_FLASH.ld ABSOLUTE)
# OpenOCD config file
get_filename_component(OPENOCD_CONFIG board/stm32f429discovery.cfg ABSOLUTE)

add_subdirectory(Projects)

# Validate that STM32CubeMX code is compatible with C standard
if(CMAKE_C_STANDARD LESS 11)
    message(ERROR "Generated code requires C11 or higher")
endif()