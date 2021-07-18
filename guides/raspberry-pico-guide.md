# Raspberry Pico projects companion

This small guide has all the hints that you require to build a hardware project using a Raspberry Pico board as a basis.

## preparation

### Install CMake (at least version 3.13), and GCC cross compiler
```
sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi libstdc++-arm-none-eabi-newlib
```
or on OSX:
```
brew install cmake
```
and install the required arm gcc plus libs, you probably also need to install Xcode (to have the latest clang compiler):
```
brew install arm-none-eabi-gcc
```

### Clone the Raspberry PICO SDK project from GIT and update submodules
```
$ clone git@github.com:raspberrypi/pico-sdk.git
$ git submodule update --init
```
Ensure that the environment var `PICO_SDK_PATH` is pointing to the folder where you installed the Raspberry pico sdk. You can do this in your profile or in front of the cmake command.

### Create a project
* Create a folder and copy the `pico_sdk_import.cmake` file from `$PICO_SDK_PATH\external` into the project folder.
* Now create a `CMakeLists.txt` in the project folder. And adjust it with the following details if you want to interact with the PICO board using the USB port.
```
cmake_minimum_required(VERSION 3.13)

# initialize pico-sdk from GIT
# (note this can come from environment, CMake cache etc)
set(PICO_SDK_FETCH_FROM_GIT on)

# pico_sdk_import.cmake is a single file copied from this SDK
# note: this must happen before project()
include(pico_sdk_import.cmake)

project(<projectname>)

# initialize the Raspberry Pi Pico SDK
pico_sdk_init()

# rest of your project
add_executable(<projectname> <projectname>.c )

pico_set_program_name(<projectname> "<projectname>")
pico_set_program_version(blinktest "0.1")

pico_enable_stdio_uart(<projectname> 1)
pico_enable_stdio_usb(<projectname> 1)

# Add the standard library to the build
target_link_libraries(<projectname> pico_stdlib)

pico_add_extra_outputs(<projectname>)
```

### Build the project
* create a build folder and change dirs to this folder
```
$ mkdir build
$ cd build
```
* build the cmake project
```
cmake ..
```
* and now build the project with make
```
make
```
You now have a binary of the project called `<projectname>.uf2`. This file you can drop in the Raskberry pico file system. To get to this filesystem. Unplug the device and then plug it in whilst pressing the little button on the board. The board will mount as if it were a USB stick. This is where you drop the `uf2` file of the project.

### Interact with the project
For this it helps to have screen installed. Then find the serial device of the board (you may need to scan the `/dev` folder to find out). When you know the device name you can use the `screen` command as follows:
```
screen /dev/tty.usbmodem14401 115200
```
**note** for my OSX environment the device appeard on `/dev/tty.usbmodem14401`.

### Example project
The hello world equivalent of an IoT project is the project that will make a led blink. This is the first project that you could save under `blinktest.c` in a project folder that I suggest you call blinktest.
```C
#include <stdio.h>
#include "pico/stdlib.h"

int main()
{
    stdio_init_all();
    const uint LED_PIN = 25;
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    while(true){
        gpio_put(LED_PIN, 1);
        printf("ON\n");
        sleep_ms(1000);
        gpio_put(LED_PIN, 0);
        printf("OFF\n");
        sleep_ms(1000);
    }

    return 0;
}
```