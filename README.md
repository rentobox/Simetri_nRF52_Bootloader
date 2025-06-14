# Adafruit nRF52 Bootloader

[![Build Status](https://github.com/adafruit/Adafruit_nRF52_Bootloader/workflows/Build/badge.svg)](https://github.com/adafruit/Adafruit_nRF52_Bootloader/actions)

A CDC/DFU/UF2 bootloader for Nordic nRF52 microcontroller. UF2 is an easy-to-use bootloader that appears as a flash drive. You can just copy `.uf2`-format application images to the flash drive to load new firmware. See https://github.com/Microsoft/uf2 for more information.

DFU via serial/CDC requires [adafruit-nrfutil](https://github.com/adafruit/Adafruit_nRF52_nrfutil), a modified version of [Nordic nrfutil](https://github.com/NordicSemiconductor/pc-nrfutil). Install `python3` if it is not installed already and run this command to install adafruit-nrfutil from PyPi:

```
$ pip3 install --user adafruit-nrfutil
```

## Supported Boards

Officially supported boards are:

- nRF52833 DK

## Features

- DFU over Serial and OTA ( application, Bootloader+SD )
- Self-upgradable via Serial and OTA
- DFU using UF2 (https://github.com/Microsoft/uf2) (application only)
- Auto-enter DFU briefly on startup for DTR auto-reset trick (832 only)

## How to use

There are two pins, `DFU` and `FRST` that bootloader will check upon reset/power:

- `Double Reset` Reset twice within 500 ms will enter DFU with UF2 and CDC support (only works with nRF52840)
- `DFU = LOW` and `FRST = HIGH`: Enter bootloader with UF2 and CDC support
- `DFU = LOW` and `FRST = LOW`: Enter bootloader with OTA, to upgrade with a mobile application such as Nordic nrfConnect/Toolbox
- `DFU = HIGH` and `FRST = HIGH`: Go to application code if it is present, otherwise enter DFU with UF2

## Burn & Upgrade with pre-built binaries

You can burn and/or upgrade the bootloader with either a J-link or DFU (serial) to a specific pre-built binary version
without the hassle of installing a toolchain and compiling the code.
This is preferred if you are not developing/customizing the bootloader.
Pre-builtin binaries are available on GitHub [releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)

Note: The bootloader can be downgraded. Since the binary release is a merged version of
both bootloader and the Nordic SoftDevice, you can freely upgrade/downgrade to any version you like.

## How to compile and build

You should only continue if you are looking to develop bootloader for your own.
You must have have a J-Link available to "unbrick" your device.

### Prerequisites

- ARM GCC
- Nordic's [nRF5x Command Line Tools](https://www.nordicsemi.com/Software-and-Tools/Development-Tools/nRF-Command-Line-Tools)
- [Python IntelHex](https://pypi.org/project/IntelHex/)

### Build:

Firstly clone this repo with following commands

```
git clone https://github.com/adafruit/Adafruit_nRF52_Bootloader
cd Adafruit_nRF52_Bootloader
git submodule update --init
```

Then build it with `make BOARD={board} all`, for example:

```
make BOARD=feather_nrf52840_express all
```

For the list of supported boards, run `make` without `BOARD=` :

```
$ make
You must provide a BOARD parameter with 'BOARD='
Supported boards are: feather_nrf52840_express feather_nrf52840_express pca10056
Makefile:90: *** BOARD not defined.  Stop
```

### Flash

To flash the bootloader (without softdevice/mbr) using JLink:

```
make BOARD=feather_nrf52840_express flash
```

If you are using pyocd as debugger, add `FLASHER=pyocd` to make command:

```
make BOARD=feather_nrf52840_express FLASHER=pyocd flash
```

To upgrade the bootloader using DFU Serial via port /dev/ttyACM0

```
make BOARD=feather_nrf52840_express SERIAL=/dev/ttyACM0 flash-dfu
```

To flash SoftDevice (will also erase chip):

```
make BOARD=feather_nrf52840_express flash-sd
```

To flash MBR only

```
make BOARD=feather_nrf52840_express flash-mbr
```

### Common makefile problems

#### `arm-none-eabi-gcc`: No such file or directory

If you get the following error ...

```
$ make BOARD=feather_nrf52840_express all
Compiling file: main.c
/bin/sh: /usr/bin/arm-none-eabi-gcc: No such file or directory
make: *** [_build/main.o] Error 127
```

... you may need to pass the location of the GCC ARM toolchain binaries to `make` using
the variable `CROSS_COMPILE` as below:
```
$ make CROSS_COMPILE=/opt/gcc-arm-none-eabi-9-2019-q4-major/bin/arm-none-eabi- BOARD=feather_nrf52832 all
```

For other compile errors, check the gcc version with `arm-none-eabi-gcc --version` to insure it is at least 9.x.

#### `ModuleNotFoundError: No module named 'intelhex'`

Install python-intelhex with

```
pip install intelhex
```

#### `make: nrfjprog: No such file or directory`

Make sure that `nrfjprog` is available from the command-line. This binary is
part of Nordic's nRF5x Command Line Tools.
