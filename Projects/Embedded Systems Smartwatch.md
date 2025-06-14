# Purpose
This project is loosely based on https://github.com/ZSWatch/ZSWatch?tab=readme-ov-file. It is a touchscreen smartwatch that has basic features like: alarm clock, timers, GPS capabilities, touch interfacing, etc.
# Operating System
ZephyrOS is an open-source real time operating system which is perfect for projects that require small yet scalable OS. In this project, I will set up the OS on an ESP32 Wroom board.

In order to begin, we need to download Zephyr for our development device OS (Windows) at https://docs.zephyrproject.org/latest/develop/getting_started/index.html
> Zephyr will also need these dependencies installed: Python, CMake, and Devicetree compiler



# Parts Used
- ESP32-PICO-D4 - 4MB flash storage
- Touchscreen display
- BMA400 Accelerometer + Pedometer
- MCP73831 LiPo Charger
- CH340E USB Serial
- Quectel L96 GPS module
- 64MB RAM (might be able to go a little less)
- microSD
- Broadcom [APDS-9306-065](https://docs.broadcom.com/docs/AV02-4755EN) Light Sensor for automatic brightness control

Maybe will use **ST LIS2MDLTR Magnetometer** for a compass feature. Might also need a dedicated PMIC like **Nordic nPM1300**. Unsure if I'll need an external crystal or if timers on ESP will suffice.
