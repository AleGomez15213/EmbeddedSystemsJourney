# Purpose
The purpose of this project is to learn:
- How do write drivers
- How are STM devices structured
- How to use I2C to communicate with gyroscopes

This project is based on a tutorial found [here](https://www.youtube.com/watch?v=_JQAve05o_0&t=1506s&ab_channel=Phil%E2%80%99sLab). The project also incorporates creating a custom PCB board for the microcontroller and accelerometer. Ideally, this accelerometer will be something I can use for future projects like the smartwatch which will  need some kind of gyroscope or acceleration sensor.

# Parts Used
- STM32F microcontroller
- ADXL355 accelerometer and temperature sensor
- Custom PCB

# Notes
In order to get the device to function, I needed too...
1) Set up MCU in STM32CubeIDE
2) Set up header file code
3) Write low level functions that can read/write data to registers
4) Initialize sensors
5) Read temperature
6) Read acceleration
## 4) Initialize sensors
One way to catch errors during initialization is to make an `int errorNum` which keeps track of anytime a register does not match the expected value. Additionally, since we are using HAL (Hardware Abstraction Layer) we can check if our registers data have an expected value by seeing if they match `HAL_OK`. Implementation example shown below

```c
status = ADXL355_WriteRegister(dev, ADXL355_REG_POWER_CTL, &regData);
errNum += (status != HAL_OK);
return errNum;
```

`ADXL355_WriteRegister` will simply return a status number. If this number is expected, no error will occur. However, if this number is not an expected value, it will raise the `errNum` int and this can be returned at the end of a function. Any number returned other than 0 means an error occurred.

## 5) Temperature Measurements
This sensor splits temperature data into 2 registers (0x06 and 0x07) where 0x06 is the high byte (TEMP2) and 0x07 is the low byte (TEMP1). **The datasheet tells us that bits 7:4 of TEMP2 are reserved meaning we only care about bits 3:0 of TEMP2 and 7:0 of TEMP1**
- As such, we can isolate these bits by using AND operation with TEMP2 and 0x0F. This will give us the lower 4 bits.
- Since TEMP2 is the MSB, we need to shift left by 8 and OR TEMP1
`uint16_t tempature = [ (TEMP2 & 0x0F) << 8 ] OR TEMP1`

Then we can just use linear mapping to convert to Celsius. The datasheet explains this...

When `tempRaw` = 1852 -> `temp_C` = 25C, therefore `temp_C = (tempRaw - 1852) + 25`. This will be our offset.

Our slope is -9.05 LSB/C so `slope` = $\frac{1}{-9.05}$. The code will look like this:

```c
dev->temp_C = -0.11049723756.0f * ( (float)tempRaw - 1852.0f ) + 25.0f
```

