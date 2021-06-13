cmake_minimum_required(VERSION 3.5)

project(untitled10 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE OFF)

# Set arduino install directory
set(ARDUINO_DIR "/home/atom/Downloads/arduino-1.8.15/hardware/arduino/avr")

set(BOOTLOADER_DIR ${ARDUINO_DIR}/bootloaders/gemma)
# Set arduino core libs directry
set(ARDUINO_LIB ${ARDUINO_DIR}/libraries)
# Set arduino external libs directory
set(ARDUINO_EXT_LIB "/home/atom/Arduino/libraries")
# Set arduino cpu
set(ARDUINO_MCU atmega2560)

# Define arduino IDE version
add_definitions(-DARDUINO=10815)
add_definitions(-DARDUINO_ARCH_AVR)
add_definitions(-DARDUINO_AVR_MEGA2560)
# Define target cpu freq
add_definitions(-DF_CPU=16000000L)
#Define target cpu
add_definitions(-D__AVR_ATmega2560__)

add_definitions(-D__PROG_TYPES_COMPAT__)
add_compile_options(-fno-rtti -fno-threadsafe-statics -fpermissive -Os)

add_compile_options(-mmcu=${ARDUINO_MCU} -MMD -fdata-sections -ffunction-sections)

#file(GLOB CORE_HEADERS
#       ${ARDUINO_DIR}/cores/arduino/*.h
#       ${ARDUINO_DIR}/variants/standard/*.h)

#message(STATUS "CORE_HEADERS: " ${CORE_HEADERS})

file(GLOB_RECURSE CORE_SOURCES
       ${ARDUINO_DIR}/cores/arduino/*.c
       ${ARDUINO_DIR}/cores/arduino/*.cpp
       ${ARDUINO_DIR}/Section/*.cpp)

#message(STATUS "CORE_SOURCES: " ${CORE_SOURCES})

file(GLOB_RECURSE LIB_SOURCES
    ${ARDUINO_LIB}/*.cpp
    ${ARDUINO_LIB}/*.c
    ${ARDUINO_LIB}/*.s)

#message(STATUS "LIB_SOURCES: " ${LIB_SOURCES})

file(GLOB_RECURSE EXT_LIB_SOURCES
    ${ARDUINO_EXT_LIB}/*.cpp
    ${ARDUINO_EXT_LIB}/*.c
    ${ARDUINO_EXT_LIB}/*.s)

#message(STATUS "EXT_LIB_SOURCES: " ${EXT_LIB_SOURCES})

add_executable(${PROJECT_NAME} ${CORE_SOURCES} ${LIB_SOURCES} ${EXT_LIB_SOURCES}  main.cpp)

target_link_options(${PROJECT_NAME} PRIVATE -mmcu=${ARDUINO_MCU} -flto -fuse-linker-plugin -Wl,--gc-sections -lc -lm)
target_include_directories(${PROJECT_NAME} PUBLIC
    ${ARDUINO_DIR}/cores/arduino/
    ${ARDUINO_DIR}/libraries/
    ${ARDUINO_DIR}/libraries/HID/src
    ${ARDUINO_DIR}/libraries/SoftwareSerial/src
    ${ARDUINO_DIR}/libraries/SPI/src
    ${ARDUINO_DIR}/libraries/Wire/src
    ${ARDUINO_DIR}/variants/standard/
    ${ARDUINO_EXT_LIB}/ArduinoRS485/src
    ${ARDUINO_EXT_LIB}/ArduinoModbus/src)


get_filename_component(COMPILER_PATH ${CMAKE_CXX_COMPILER} DIRECTORY)
message(STATUS "Compiler path: " ${COMPILER_PATH})

add_custom_command(
  TARGET ${PROJECT_NAME}
  COMMAND ${COMPILER_PATH}/avr-size ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}
                   --mcu=${ARDUINO_MCU}
                   -C --format=avr
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}
  VERBATIM)

add_custom_command(
  OUTPUT OBJCOPY_HEX
  COMMAND ${COMPILER_PATH}/avr-objcopy
                    -O ihex
                    -R .eeprom
                    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}
                    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}
  VERBATIM)

add_custom_command(
  TARGET ${PROJECT_NAME}
  COMMAND ${COMPILER_PATH}/avr-objcopy
                    -O binary
                    -S -g
                    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}
                    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}
  VERBATIM)

add_custom_command(
  TARGET ${PROJECT_NAME}
  COMMAND ${COMPILER_PATH}/avrdude
                    -c wiring
                    -p m2560
                    -P /dev/ttyUSB0
                    -b 115200
                    -U flash:w:${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex:i
                    -C ${BOOTLOADER_DIR}/avrdude.conf
                    -D
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.bin
  VERBATIM)

