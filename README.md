# Arduino template

A template which allows developers to create, build, and upload arduino based projects while using other c++ IDEs and have access to c++ LSPs for code autocompletion, warnings, and gotos.

## Usage

Clone or download the repository

```sh
git clone https://github.com/Cristi104/Arduino-template && cd Arduino-template
```

Now you nedd to configure the CMake variables in special the board and the path to arduino-cli.
By default the source code is compiled and uploaded for a Adruino Uno R3.

```CMake
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
```

Now that you have changed the board and cli directory set correctly you can build and upload.

```sh 
cmake -B build -S . && cmake --build build --target upload
```


