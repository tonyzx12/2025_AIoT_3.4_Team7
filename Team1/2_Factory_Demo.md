# ESP32-P4-EYE Factory Demo Build and Flash Guide

This guide covers building and flashing the factory demo firmware for ESP32-P4-EYE vision development board, which restores the device to its original factory state with all demo features.

Table of Contents:
- [Build Process (On AWS SageMaker/Linux)](#build-process-on-aws-sagemakerlinux)
- [Flash Process (On Windows PC)](#flash-process-on-windows-pc)
- [Verify Installation](#verify-installation)


## Build Process (On AWS SageMaker/Linux)
### 1. Setup ESP-IDF Environment
```bash
# Navigate to ESP-IDF directory
cd ~/SageMaker/esp/esp-idf

# Install ESP-IDF tools
./install.sh all

# Setup environment variables
source ./export.sh
```
### 2. Clone ESP Dev Kits Repository (One-time Only)
```bash
# Clone the repository with submodules
git clone --recursive https://github.com/espressif/esp-dev-kits.git
```
Note: This step only needs to be done once. Skip if already cloned.

### 3. Checkout Specific ESP-IDF Version
```bash
# Navigate to ESP-IDF directory
cd ~/SageMaker/esp/esp-idf

# Checkout release v5.5 branch
git checkout release/v5.5

# Checkout specific commit
git checkout 98cd765953dfe0e7bb1c5df8367e1b54bd966cce
```
### 4. Apply Required Patches
The factory demo requires two patches to fix SPI clock source and SDMMC buffer issues:
```shell
# Navigate to factory demo directory
cd ../esp-dev-kits/examples/esp32-p4-eye/examples/factory_demo

# Copy patch files to ESP-IDF directory
cp 0004-fix-spi-default-clock-source.patch ~/SageMaker/esp/esp-idf/
cp 0004-fix-sdmmc-aligned-write-buffer.patch ~/SageMaker/esp/esp-idf/

# Navigate back to ESP-IDF directory
cd ~/SageMaker/esp/esp-idf

# Apply patches
git apply 0004-fix-spi-default-clock-source.patch
git apply 0004-fix-sdmmc-aligned-write-buffer.patch
```

### 5. Configure and Build Project
```bash
# Navigate to factory demo directory
cd ../esp-dev-kits/examples/esp32-p4-eye/examples/factory_demo

# Configure project settings (select v0.0 hardware version when prompted)
idf.py menuconfig

# Set target chip to ESP32-P4
idf.py set-target esp32p4

# Build the project
idf.py build
```
> ℹ️ Configuration Notes  
During idf.py menuconfig, ensure you select hardware version v0.0 for ESP32-P4-EYE  
This matches your board's silicon revision

### 6. Build Output
After successful build, you'll see:
```bash
Project build complete. To flash, run:
 idf.py flash
or
 python -m esptool --chip esp32p4 -b 460800 --before default_reset --after hard_reset write_flash --flash-mode dio --flash-size 16MB --flash-freq 40m 0x2000 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin 0x10000 build/factory_demo.bin
```
You can find the build artifacts in the `build/` directory. 
> ℹ️ Important  
 Note the flash parameters from your build output, especially:
--flash-size 16MB (larger than hello_world due to demo features)
Memory addresses (0x2000, 0x8000, 0x10000)

### 7. Download Binary Files
Download these three files from the build/ directory:
- build/bootloader/bootloader.bin - Bootloader image
- build/partition_table/partition-table.bin - Partition table
- build/factory_demo.bin - Factory demo application firmware


## Flash Process (On Windows PC)
### 1. Install esptool
```bash
pip install esptool
```

### 2. Identify Serial Port  
Connect ESP32-P4 board via USB  
Check Device Manager to identify COM port (e.g., COM7)

### 3. Flash Firmware
Navigate to the directory containing the downloaded .bin files, then run:
```bash
python -m esptool -p COM7 --chip esp32p4 -b 115200 --no-stub --before default_reset --after hard-reset write_flash --flash-mode dio --flash-size 16MB --flash-freq 80m 0x2000 bootloader.bin 0x8000 partition-table.bin 0x10000 factory_demo.bin
```
- Adjust -p COM7 to your actual COM port
- --flash-size 16MB - Factory demo requires larger flash than hello_world
- --flash-freq 80m - Used for flashing (build suggests 40m for runtime)
- Flash addresses match the build output



## Verify Installation
Start and connect to the ESP32-P4-EYE, you should see the factory demo running with camera and display features.