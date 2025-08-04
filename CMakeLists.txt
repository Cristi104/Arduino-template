cmake_minimum_required(VERSION 3.20)

set(CMAKE_CXX_STANDARD 11)

# config settings
set(SKETCH_NAME "Arduino-template") # project name also change the root_dir name and the .ino name to the same thing

set(ARDUINO_CLI_DIR "$ENV{HOME}/.arduino15") # path to the .arduino15 directory
set(ARDUINO_DIR "$ENV{HOME}/Arduino") # path to the Arduino directory 

# you may need to change these depending on your specific compiler and arhitecture
set(ARHITECTURE "avr")
set(ARHITECTURE_VERSION "1.8.6")
set(COMPILER "avr-gcc")
set(COMPILER_VERSION "7.3.0-atmel3.6.1-arduino7")

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
file(GLOB_RECURSE include_dirs  
  LIST_DIRECTORIES true "${ARDUINO_CLI_DIR}/packages/arduino/hardware/${ARHITECTURE}/${ARHITECTURE_VERSION}/libraries/*/")
set(INCLUDE_DIRS "")
foreach(dir ${include_dirs})
  cmake_path(GET dir FILENAME filename)
  if(IS_DIRECTORY "${dir}" AND filename STREQUAL "src")
    list(APPEND INCLUDE_DIRS "${dir}")
  endif()
endforeach()

file(GLOB_RECURSE include_dirs
  LIST_DIRECTORIES true "${ARDUINO_DIR}/libraries/*")
foreach(dir ${include_dirs})
  if(IS_DIRECTORY "${dir}")
    list(APPEND INCLUDE_DIRS "${dir}")
  endif()
endforeach()
list(APPEND INCLUDE_DIRS ${CMAKE_SOURCE_DIR}/include)
list(APPEND INCLUDE_DIRS ${ARDUINO_CLI_DIR}/packages/arduino/hardware/${ARHITECTURE}/${ARHITECTURE_VERSION}/cores/arduino)
list(APPEND INCLUDE_DIRS ${ARDUINO_CLI_DIR}/packages/arduino/hardware/${ARHITECTURE}/${ARHITECTURE_VERSION}/variants/standard)
list(APPEND INCLUDE_DIRS ${ARDUINO_CLI_DIR}/packages/arduino/tools/${COMPILER}/${COMPILER_VERSION}/${ARHITECTURE}/include)

file(WRITE "${CMAKE_BINARY_DIR}/dummy.cpp" "// dummy source\n")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_executable(dummy "${CMAKE_BINARY_DIR}/dummy.cpp")
set_target_properties(dummy PROPERTIES EXCLUDE_FROM_ALL TRUE)
target_include_directories(dummy PRIVATE ${INCLUDE_DIRS})

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

