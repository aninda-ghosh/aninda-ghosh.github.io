# nRF52-Series-DFU

## Project Description

Preparing nRF52832 for OTA.

OTA is mandatory in today's days when the deadlines are near and software has to be rolled out with future proofing capabilities.

With MBED OS I didn't find very well explained example for implementing DFU, and for any new developer with a constrained time will love to use MBED os for their development with minimal effort in setting up the device drivers and focus more on the application. Hence this project is targeted to those people explicitly.

## Prerequisites

- linux system (debian based) (Can use windows as well, Since I am more comfortable with linux for these kinds of development I stick to Linux)
- mbed-cli installed
- nrgjprog installed **(Comes with the nRF Utils package)**
- nrfutil installed
- mergehex installed
- SDK used here is 15.0.0 for nRF52 Series **(Provided in th eproject folder)**
- uECC is needed since we will be using secure bootloader for our OTA purpose. We will need to build from source and GNU ARM toolchain version **gcc-arm-none-eabi-6-2017-q2-update**
  - Benefits:
    - Person having the secure private key can only push the DFU package.
    - Bootloader won't accept any other package with a different signature.
- MBED OS 5.15.6 **(Provided in the example as well)**
- Softdevice used in the given example is **s132 ver 6.0.0**


## Theory of operation

In theory the bootloader sits the very beginning of the memory that is accessed by the Instruction pointer.

A brief memory map showing the memory segments

```
 ________________________   Memory Start 
|                        |  
|       Bootloader       |  (Preparing the Bootloader from the nRF52 SDK)
|________________________|
|                        |
|                        |
|                        |
|                        |
|      Application       |
|                        |  (Preparing the application with MBED OS 5)
|                        |
|                        |
|                        |
|________________________|
|                        |  
|                        |
|       Softdevice       |  (Using .hex file of the S132 softdevice for nRF52DK)
|                        |
|________________________|  
                            Memory End
```

Bootloader broadcasts the DFU service which in terms accepts a presigned DFU package from the Device it is communicating with e.g. Mobile device.

For this example I am using nRF connect app for pushing the package.

We will tailor suit the bootloader according to our needs like:

- Change the name being broadcasted by the DFU. I am using the DFU as my name along with the BD address taken from the FICR register of the nRF for a Consistent  identifier for a nRF Chip.

## Setup for bootloader

- Get the latest SDK 15.0.0.
- Generate the key that you want to use for securing your packages using nrfutil. 
  ```
    nrfutil keys generate private.key
    nrfutil keys display --key pk --format code private.key --out_file public_key.c
  ```
- Copy the file contents into **dfu_public_key.c** file under the examples folder.
- Let's change the stuffs we need.
  - Let's broadcast the FICR Register BD address **File: nrf_dfu_ble.c**
    ```
     {
        /******************
         * CHANGED HERE
         ******************/
        
        /**
         * using the FICR Register data to set the local name. It will be used to
         * identify the DFU target after triggering the DFU dynamically.
         */
        
        NRF_LOG_DEBUG("Using custom advertising name");
        ble_gap_addr_t addr;
        err_code = sd_ble_gap_addr_get(&addr);
        char *ptr = NRF_DFU_BLE_ADV_NAME;
        for(; name_len < strlen(NRF_DFU_BLE_ADV_NAME); name_len++){
            device_name[name_len] = *ptr;
            ptr++;
        }
        device_name[name_len++] = '[';
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[1]>>8);
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[1]);
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[0]>>24);
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[0]>>16);
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[0]>>8);
        device_name[name_len++] = (uint8_t)(NRF_FICR->DEVICEADDR[0]);
        device_name[name_len++] = ']';
    }
    ```
- Build the secure bootloader program in the DFU example using make command (Make sure you have **gcc-arm-none-eabi-9-2019-q4-major** toolchain installed)
    ```
    make 
    ```
- Once the build is done we will find the .hex file for our secured bootloader under _build folder.

## Setup for the application code

- We will create a service and characteristics for DFU in our application and will use the same to trigger the underlying bootloader.
- We have added an extra layer of security by adding an alphanumeric key which is needed to be sent over encrypted BLE channel to trigger the DFU.
- Once the application receives the DFU trigger it then writes into **NRF_POWER->GPREGRET** which is non-volatile in nature and resets the system.
- Next time when the bootloader check that particular bit it gets into the DFU mode for upgrading.
- Currently this is a single bank implementation and if the upgrade process get's interrupted the device stays in bootloader mode until a valid application image is pushed.

## Preparation for the DFU Package

- First the Softdevice and the Generated Bootloader is merged into a single hex file for flashing using **mergehex**.
- Or the application can also be flashed after generating the bootloader settings page and merging it with Application.hex, Bootloader.hex, Softdevice.hex.
    ```
    nrfutil settings generate --family NRF52 --application $application --application-version 0 --bootloader-version 0 --bl-settings-version 1 bootloader_setting.hex

    mergehex --merge s132_nrf52_6.0.0_softdevice.hex nrf52832_xxaa_s132.hex --output bootloader_softdevice.hex
    ```
- Flash the merged files onto the device for starting the OTA supported application.

## Additional support materials

It's always better to have relevant documents while developing hence this section.

- Memory Regions for segmented development:
    <https://devzone.nordicsemi.com/nordic/short-range-guides/b/getting-started/posts/adjustment-of-ram-and-flash-memory>
- DFU bootloader development:
    <https://devzone.nordicsemi.com/nordic/short-range-guides/b/software-development-kit/posts/getting-started-with-nordics-secure-dfu-bootloader>
- Memory optimizations: 
    <https://os.mbed.com/docs/mbed-os/v5.14/tutorials/optimizing.html>
    
    ```
    "*": {
        "platform.memory-tracing-enabled": true
    }
    ```