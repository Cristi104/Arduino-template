cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)

# config settings
set(SKETCH_NAME "arduino-template") # project name also change the root_dir name and the .ino name to the same thing

set(BOARD_TYPE "avr") # microprocessor arhitecture
set(BOARD_TYPE_VERSION "1.8.6") # arhitecture version
set(ARDUINO_CLI_DIR "~/snap/arduino-cli/60/.arduino15/packages/arduino/hardware") # path to the hardware folder inside arduino-cli for headers

# set for normal board upload
set(ARDUINO_BOARD "arduino:avr:uno")
set(SERIAL_PORT "/dev/ttyUSB0")

# set for upload via programmer upload with avrdude
# if you use non avr based boards you need to change the upload command 
set(PROGRAMMER "usbasp")
set(MICRO_CONTROLLER "m328p")

# end of config

project(${SKETCH_NAME} C CXX)

set(CMAKE_BUILD_DIR ${CMAKE_SOURCE_DIR}/build)
set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build/bin)

# this is just a hack to get the compile_commands.json file with the good include paths for LSPs
file(WRITE "${CMAKE_BINARY_DIR}/dummy.cpp" "// dummy source\n")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_executable(dummy "${CMAKE_BINARY_DIR}/dummy.cpp")
set_target_properties(dummy PROPERTIES EXCLUDE_FROM_ALL TRUE)
target_include_directories(dummy PRIVATE
   ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_BUILD_DIR}/lib/*/src
    ${CMAKE_BUILD_DIR}/lib/*/include
    ${ARDUINO_CLI_DIR}/${BOARD_TYPE}/${BOARD_TYPE_VERSION}/cores/arduino
    ${ARDUINO_CLI_DIR}/${BOARD_TYPE}/${BOARD_TYPE_VERSION}/variants/standard
    /usr/lib/avr/include
)

add_custom_target(compile
    COMMAND arduino-cli compile --fqbn ${ARDUINO_BOARD} --build-path ${CMAKE_BINARY_DIR}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

add_custom_target(upload
    COMMAND arduino-cli upload -p ${SERIAL_PORT} --fqbn ${ARDUINO_BOARD} --input-dir ${CMAKE_BINARY_DIR}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    DEPENDS compile
)

add_custom_target(upload_programmer
    COMMAND avrdude -c ${PROGRAMMER} -p ${MICRO_CONTROLLER} -U flash:w:build/bin/${SKETCH_NAME}.ino.hex:i
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    DEPENDS compile
)

