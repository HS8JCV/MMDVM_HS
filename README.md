# Introduction

This is the source code of ZUMspot/MMDVM_HS, personal hotspot (ADF7021 version of the MMDVM firmware), based on Jonathan G4KLX's [MMDVM](https://github.com/g4klx/MMDVM) software. This firmware supports D-Star, DMR, System Fusion and P25 digital modes.

This software is intended to be run on STM32F103 microcontroller. Also, Arduino with 3.3 V I/O (Arduino Due and Zero) and Teensy (3.1, 3.2, 3.5 or 3.6) are supported. You can build this code using Arduino IDE with Roger Clark's [STM32duino](https://github.com/rogerclarkmelbourne/Arduino_STM32/tree/ZUMspot) package, or using command line tools with ARM GCC tools. The preferred method under Windows is using STM32duino, and under Linux or macOS (command line) is using [STM32F10X_Lib](https://github.com/juribeparada/STM32F10X_Lib). 

This software is licenced under the GPL v2 and is intended for amateur and educational use only. Use of this software for commercial purposes is strictly forbidden.

# Features

- Supported modes: D-Star, DMR, Yaesu Fusion and P25 Phase 1
- Automatic mode detection (scanning)
- G4KLX software suite: [MMDVMHost](https://github.com/g4klx/MMDVMHost), [ircDDBGateway](https://github.com/dl5di/OpenDV), [YSFGateway](https://github.com/g4klx/YSFClients), [P25Gateway](https://github.com/g4klx/P25Clients) and [DMRGateway](https://github.com/g4klx/DMRGateway)
- Bands: 144, 220, 430 and 900 MHz (VHF requires external inductor)
- Status LEDs (PTT, COR and digital modes)
- Serial repeater port for Nextion displays
- Support for native USB port in STM32F103 CPU
- RSSI support
- CW ID support
- Full duplex support with two ADF7021

# Important notes

The ADF7021 (or RF7021SE module) must operate with a 14.7456 MHz TCXO and with at least 2 ppm of frequency stability. You could use also 12.2880 MHz TCXO, but this frequency configuration has less testing. Any other TCXO frequency is not supported.

The jumper wire to CLKOUT in RF7021SE module is not longer required for lastest MMDVM_HS firmware. But CE pin connection of ADF7021 is required for proper operation of ZUMspot.

Be aware that some Blue Pill STM32F103 board are defectives. If you have trouble with USB, please check this [link](http://wiki.stm32duino.com/index.php?title=Blue_Pill).

VHF (144-148 MHz) support for ZUMSpot is added by an external 18 nH inductor between L1 and L2 pins of ADF7021. This will enable dual band (VHF/UHF) operation.

Dual ADF7021 for full duplex operation (#define DUPLEX in Config.h) will work only with a big RX/TX frequency separation (5 MHz or more in UHF band for example) and proper antenna filtering. At the moment #define LIBRE_KIT_ADF7021 (Config.h) with STM32F103 platform is supported. Please see [BUILD.md](BUILD.md) for pinout details.

If you can't decode any 4FSK modulation (DMR, YSF and P25) with your ZUMspot, the common solution is to adjust RX frequency offset (RXOffset) in your MMDVM.ini file. Please try with steps of +-100 Hz until you get low BER. If you don't have test equipment, the only procedure is trial and error. In some cases TXOffset adjustment is also required for proper radio decoding. If you have test equipment, enable TEST_TX feature (see "Hidden functions" in [BUILD.md](BUILD.md) document).

# Quick start

Please see [BUILD.md](BUILD.md) for more details, and also [MMDVM](https://groups.yahoo.com/neo/groups/mmdvm/info) Yahoo Groups. You also can check at MMDVM_HS/scripts folder for some automatic tasks.

## Binary firmware installation

Please check the latest firmware [here](https://github.com/juribeparada/MMDVM_HS/releases).

### Windows

Download the ZUMspotFW firmware upgrade utility (ZUMspotFW_setup.exe) from the [releases section](https://github.com/juribeparada/MMDVM_HS/releases/download/v1.0.2/ZUMSpotFW_setup.exe).

This utility includes firmwares binaries and USB drivers for Windows 7/8/10. If you have problems with the installer, you can download [ZUMspotFW.zip](https://github.com/juribeparada/MMDVM_HS/releases/download/v1.0.2/ZUMspotFW.zip) for a manual installation.

### Linux or macOS

Download the script (*.sh) that matches with your ZUMspot/MMDVM_HS board:

- install_fw_rpi.sh: only for ZUMspot RPi board
- install_fw_hshat.sh: only for MMDVM_HS_Hat board
- install_fw_librekit.sh: only for ZUMspot Libre Kit board or generic MMDVM_HS board
- install_fw_usb.sh: only for ZUMspot USB dongle
- install_fw_duplex.sh: only for MMDVM_HS with dual ADF7021

For example, download the ZUMspot RPi upgrade script:

    cd ~
    curl -OL https://raw.github.com/juribeparada/MMDVM_HS/master/scripts/install_fw_rpi.sh

make the script executable:

    chmod +x install_fw_rpi.sh

stop your MMDVMHost process and run (you will need the root password):

    ./install_fw_rpi.sh

and wait to complete the upgrading process.

## Build from the sources

You could check at MMDVM_HS/configs for common Config.h examples, using different ZUMspot boards.

### MMDVM_HS_Hat

Please visit Mathis Schmieder GitHub [here](https://github.com/mathisschmieder/MMDVM_HS_Hat/blob/master/README.md) for detailed instructions.

### ZUMspot Libre Kit (under Linux Raspbian)

If you are using Pi-Star, expand filesystem (if you haven't done before):

    sudo pistar-expand
    sudo reboot

Enable RW filesystem if you are using Pi-Star:

    rpi-rw

Install the necessary software tools:

    sudo apt-get update
    sudo apt-get install gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi

Download the sources:

    cd ~
    git clone https://github.com/juribeparada/MMDVM_HS
    cd MMDVM_HS/
    git clone https://github.com/juribeparada/STM32F10X_Lib

Build the firmware with USB bootloader support (the default Config.h is OK for Libre Kit):

    make bl

If you are using Pi-Star, stop services:

    sudo pistar-watchdog.service stop
    sudo systemctl stop mmdvmhost.timer
    sudo systemctl stop mmdvmhost.service

Upload the firmware to ZUMspot Libre Kit using the USB port (your Libre Kit has the bootloader already installed):

    sudo make dfu devser=/dev/ttyACM0

### ZUMspot RPi

#### Enable serial port in Raspberry Pi 3 or Pi Zero W

This this necessary only if you are installing a fresh copy of Raspbian OS.

Edit /boot/cmdline.txt:

    sudo nano /boot/cmdline.txt
    (remove the text: console=serial0,115200)

Disable services:

    sudo systemctl disable serial-getty@ttyAMA0.service
    sudo systemctl disable bluetooth.service

Edit /boot/config.txt

    sudo nano /boot/config.txt

and add the following lines at the end of /boot/config.txt:

    enable_uart=1
    dtoverlay=pi3-disable-bt

Reboot your RPi:

    sudo reboot

#### Build the firmware and upload to ZUMspot RPi

If you are using Pi-Star, expand filesystem (if you haven't done before):

    sudo pistar-expand
    sudo reboot

Enable RW filesystem if you are using Pi-Star:

    rpi-rw

Install the necessary software tools:

    sudo apt-get update
    sudo apt-get install gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi
    
    cd ~
    git clone https://git.code.sf.net/p/stm32flash/code stm32flash
    cd stm32flash
    make
    sudo make install

Download the firmware sources:

    cd ~
    git clone https://github.com/juribeparada/MMDVM_HS
    cd MMDVM_HS/
    git clone https://github.com/juribeparada/STM32F10X_Lib

Edit Config.h

    nano Config.h
    
and enable:

    #define ZUMSPOT_ADF7021
    #define ENABLE_ADF7021
    #define ADF7021_14_7456
    #define STM32_USART1_HOST
    #define ENABLE_SCAN_MODE

Build the firmware:

    make

If you are using Pi-Star, stop services:

    sudo pistar-watchdog.service stop
    sudo systemctl stop mmdvmhost.timer
    sudo systemctl stop mmdvmhost.service

Upload the firmware to ZUMspot RPi board:

    sudo make zumspot-pi

