# Capabilities of STM32L152RCT6

## Project Description

This repository is intended to provide a boilerplate for the STM32 MCU with some common and some uncommon functionalities.

[Project Page](https://github.com/aninda-ghosh/fun-with-stm32)

## Outcomes Expected
- Basic GPIO Level implentations for understanding what is happening.
- Use the peripherals available in the STM MCu itself like RTC Calendar, RTC Backup Registers, ADC, DAC etc.
- Setup of Communication protocols like I2C, SPI, USART etc. [EEPROM RTC Branch](https://github.com/aninda-ghosh/fun-with-stm32/tree/eeprom-rtc)
    See commits
    ```
    2f7d9bf Added I2C for EEPROM
    80a6c2d Added EEPROM Partitioning
    ```
- Use I2C and SPI to implement communications between different kinds of sensors in real life scenarios e.g. MPU9250 9DOF MPU. [MPU Branch](https://github.com/aninda-ghosh/fun-with-stm32/tree/MPU9250)

- Dive into the Low Power Modes and Touch Sensing available in the STM32L series. [Touch Sensing Branch](https://github.com/aninda-ghosh/fun-with-stm32/tree/touch-sensing)