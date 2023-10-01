# Multiboot System in STM32

## Project Description

This project is a demo project of accomodating multiple boot sects in an ARM 32 SOC. 

[Project Page](https://github.com/aninda-ghosh/STM32-MultiBoot)


## Prior System Requirements
- Three Sectors (App1, App2, & Bootloader)
- Added 1 Booloader and 2 Application Project for sideloading via STLINK.

## File and Folder Structure

```
├── App1-L1Discovery
│   ├── Core
│   │   └── Src #Source Files
│   └── STM32L152RCTX_FLASH.ld
├── App2-L1Discovery
│   ├── Core
│   │   └── Src #Source Files
│   └── STM32L152RCTX_FLASH.ld
└── Bootloader-L1Discovery
    ├── Core
    │   └── Src #Source Files
    └── STM32L152RCTX_FLASH.ld
```
