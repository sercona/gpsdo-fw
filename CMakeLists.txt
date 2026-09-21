cmake_minimum_required(VERSION 3.22)

# Setup compiler settings
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)


# Define the build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()

# Set the project name
set(CMAKE_PROJECT_NAME gpsdo)

# Include toolchain file
include("cmake/gcc-arm-none-eabi.cmake")

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

# Enable CMake support for ASM and C languages
enable_language(C ASM)

# Core project settings
project(${CMAKE_PROJECT_NAME})
message("Build type: " ${CMAKE_BUILD_TYPE})

# Define EEPROM_AUTO_SAVE
if(NOT DEFINED EEPROM_AUTO_SAVE)
  set(EEPROM_AUTO_SAVE "0")
endif()

if(EEPROM_AUTO_SAVE STREQUAL "1")
  add_compile_definitions(EEPROM_AUTO_SAVE=1)
elseif(EEPROM_AUTO_SAVE STREQUAL "0")
  add_compile_definitions(EEPROM_AUTO_SAVE=0)
else()
  message(FATAL_ERROR "EEPROM_AUTO_SAVE must be 0 or 1")
endif()


add_subdirectory(lib)

# Create an executable object type
add_executable(${CMAKE_PROJECT_NAME})

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    src/main.c
    src/eeprom.c
    src/frequency.c
    src/gps.c
    src/int.c
    src/menu.c
)


# Add include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
)

# Add linked libraries
target_link_libraries(${CMAKE_PROJECT_NAME}
    stm32cubemx
    eeprom
    LCD
)

# Generate .bin from .elf after build
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND arm-none-eabi-objcopy -O binary
        $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
        ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.bin
    COMMENT "Generating gpsdo.bin"
)

# Flash target
set(OPENOCD_ROOT "$ENV{HOME}/.platformio/packages/tool-openocd"
    CACHE PATH "Path to OpenOCD root")
set(OPENOCD_BIN   "${OPENOCD_ROOT}/bin/openocd")
set(OPENOCD_SCRIPTS "${OPENOCD_ROOT}/scripts")

add_custom_target(flash
    DEPENDS ${CMAKE_PROJECT_NAME}
    COMMAND ${OPENOCD_BIN}
        -s ${OPENOCD_SCRIPTS}
        -f interface/stlink.cfg
        -f target/stm32f1x.cfg
        -c "program ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.bin verify reset exit 0x08000000"
    COMMENT "Flashing ${CMAKE_PROJECT_NAME}.bin via ST-Link"
)
