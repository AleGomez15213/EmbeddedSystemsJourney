# Introduction
The goal of this project is to learn the basics of using Zephyr as an RTOS. FreeRTOS is an easier alterative which has less of a learning curve but it seems more real world applications will use RTOS similar to Zephyr. The goal of this projects are...
- Set up Zephyr on Windows 11 machine
- Build ZephyrOS and flash
- Build Blinky program
- Set up user input
- Set up GUI
- Interact with Zephyr application using touch screen
The idea is that Zephyr will be used in a variety of other projects I do including the embedded systems smartwatch. This first note will only cover a basic example of blinking an LED. The above goals will be continued in later notes.
# Set Up
Zephyr took a long time to set up on my computer. One important thing is that filepaths with spaces can impede installing and running the zephyr apps. For example:
`C:\Users\John Doe` can eventually cause issues when attempting to build zephyr applications. I had to reset my computer because I didn't want to risk changing the folder name to remove the space as this can sometimes cause other issues. Thankfully, it was not much of a hassle for me to do this on my machine.

1. Windows uses winget in order to install the pre-req:
	`winget install Kitware.CMake Ninja-build.Ninja oss-winget.gperf python Git.Git oss-winget.dtc wget 7zip.7zip`
2. Zephyr relies on python dependencies to work and will use west as its meta tool
```powershell
	cd $Env:HOMEPATH
	python -m venv zephyrproject\.venv
	zephyrproject\.venv\Scripts\Activate.ps1 # activates the venv
```
3. Install west and get source code
```powershell
	pip install west
	west init zephyrproject
	cd zephyrproject
	west update
	west zephyr-export
	west packages pip --install
```
4. Install the Zephyr SDK. This step requires the installation of 7-zip. Something I did not initially realize was that in order to allow west to use 7zip, I had to actually add 7zip to PATH. In Windows, this can be done copying and pasting the filepath of the `7zip.exe` file. Note that the filename should not be included in the path.
```powershell
	cd $Env:HOMEPATH\zephyrproject\zephyr
	west sdk install
```

These steps complete the setup process for Zephyr. I continuously encountered many errors while undergoing the installation process. Some occurred because I didn't read specific lines in the instructions, other errors required intuition and a little bit of researching to get past.

# Building Blinky
The Blinky program I use is based of of this [tutorial series](https://www.youtube.com/watch?v=Z_7y_4O7yTw&t=190s). Note that directory structure is very important when working with Zephyr. Based on my very small research of other boards, it seems some boards require a different structure but at least for me, the strucutre I needed is below:
```
~/
|--- zephyrproject
|    |--- zephyr
|    |--- helloworld
|    |    |--- CMakeLists.txt
|    |    |--- src
|    |    |    |--- main.c
```

## CMakeLists.txt
Zephyr uses CMake as its build system generator. It's in charge of building the files for a specified board. The file I used is shown below:
```cmake
cmake_minimum_required(VERSION 3.22)
set(BOARD esp32_devkitc/esp32/procpu) 

find_package(Zephyr)
project(hello_world_blinky)

target_sources(app PRIVATE src/main.c)
```
The tutorial I followed specified `set(BOARD esp32) ` however, Zephyr folder and files change very quickly as it gets more updates. When I went to build the actual application, it gave me an error and said that `esp32` is not a specified board. I managed to find a list of available boards for build and landed on `esp32_devkitc/esp32/procpu`.
## main.c
This is the file where the main program lives. For testing purposes, it contains very little code. Initially, I just wanted to make sure I could build the app using west and then worried about implementation of features.
```c
#include <zephyr/kernel.h>
void main(void) {
    while (true) {
        k_msleep(1000);
    }
}
```
Again, Zephyr's file structure and names change very quickly. Turns out `zephyr.h` is replaced with `kernel.h`.

## Building
To build the application, we use
```powershell
west build -b esp32_devkitc/esp32/procpu -p always
```
While the tutorial was able to simply pass esp32 as the board, it seems Zephyr no would prefer you specify which CPU you would like to build for. I just arbitrarily chose procpu but might change it later depending on what is required by the project.

## Testing
Once the application has been built, we can flash the ESP32 chip

# Running Blinky
In the example above, we wrote a simple program that simply goes to sleep infinitely until it's stopped. Now we would like to actually run some code on it. Before we get into coding, we need some knowledge of how Zephyr works.
## Device
In zephyr, we are able to communicate with GPIO devices by using a `struct device*` and passing it as an argument. The ESP32 board has two GPIO devices (GPIO0, GPIO1) that we can call in order to communicate with the GPIO pins themselves. Zephyr will use a devicetree in order to hierarchically describe the capabilities of a board and, in this case, communicate with the GPIO devices. A simplified diagram of the device tree can be seen below:
```dtsi
/ {
	soc {
		gpio: gpio {
			gpio0: gpio0@3ff44000 {
				compatible = "espressif,esp32-gpio";
				reg = <0x3ff44000 0x800>;
				ngpios = <32>;
			};

			gpio1: gpio0@3ff44800 {
				compatible = "espressif,esp32-gpio";
				reg = <0x3ff44800 0x800>;
				ngpios = <8>;
			};
		}
	}
}
```

The devicetree has a `/` as the root node, and then sub-nodes of sub-nodes. In order to access `gpio0`, we'll want to use the path `/soc/gpio/gpio@3ff44000` where `3ff44000` is the ESP32 memory address or unit address. This address can be confirmed in the [Technical Manual](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf) but is found by in the devicetree file generated by zephyr. `gpio0` and `gpio1` are used to reference these devices and are called node labels. We can use these labels to get pointers to the GPIO controller. Also note that the first controller, `gpio0`, controls the first 32 pins (pins 0-31) and the second one controls the remaining 8 (pins 32-39). Therefore, a script can be written to blink an LED:
```c
#include <zephyr/kernel.h>
#include <zephyr/drivers/gpio.h>
#include <zephyr/device.h>

static const struct device *gpio_ct_dev = DEVICE_DT_GET(DT_NODELABEL(gpio0));

void main(void) {
	if (!device_is_ready(gpio_ct_dev)) {
		return;
	}

	int ret;
	ret = gpio_pin_configure(gpio_ct_dev, 25, GPIO_OUTPUT_ACTIVE);
	if (ret != 0) {
		return;
	}
    
	while (true) {
		ret = gpio_pin_set_raw(gpio_ct_dev, 25, 1);
		if (ret != 0) {
			return;
		}
		k_msleep(1000);
	
		ret = gpio_pin_set_raw(gpio_ct_dev, 25, 0);
		if (ret != 0) {
			return;
		}
		k_msleep(1000);
	}
}
```

## Code Explanation
```c
#include <zephyr/kernel.h>
#include <zephyr/drivers/gpio.h>
#include <zephyr/device.h>
```
These are the Zephyr libraries to be included. `kernel.h` will provide access to general kernel services in Zephyr. Functions like `k_msleep()`, `k_mutex()`, etc. are derived from this header file. `gpio.h` provides abstractions for configuring and interacting with GPIO pins. `drivers.h` is used to access I2C, UART, and other devices in a convenient way.

```c
static const struct device *gpio_ct_dev = DEVICE_DT_GET(DT_NODELABEL(gpio0));
```
This variable will serve as the pointer to our GPIO device (specifically gpio0). `DEVICE_DT_GET()` is a macro used to retrieve a pointer to a device struct defined in the Device Tree. We pass in gpio0 as a node. `DT_NODELABEL(gpio0)` will get the **node ID** of a device labeled `gpio0` in the device tree (See above for how device tree is structured.) We can now use `gpio_ct_dev` when changing an GPIO pin's state.

```c
if (!device_is_ready(gpio_ct_dev)) {
	return;
}
```
We first check if `gpio_ct_dev` is ready. If it isn't, exit the program.

```c
int ret;
ret = gpio_pin_configure(gpio_ct_dev, 25, GPIO_OUTPUT_ACTIVE);
if (ret != 0) {
	return;
}
```
In order to configure pin 25 as output, we then declare an integer called `ret`. This will serve as our status code. A value of 0 means successful configuration of the specified pin. Anything else can be considered an error. The function takes in a device pointer, pin number, and flags. Anytime we want to verify the validity of a configuration, we can reuse this `ret` variable.

```c
ret = gpio_pin_set_raw(gpio_ct_dev, 25, 1);
if (ret != 0) {
	return;
}
k_msleep(1000);

ret = gpio_pin_set_raw(gpio_ct_dev, 25, 0);
if (ret != 0) {
	return;
}
k_msleep(1000);
```
The code above will use `gpio_pin_set_raw()` to set the actual voltage of the pin. It takes in a device struct, pin number, and the value. A value of 1 is digital high, and 0 is digital low. This full code will cause the LED to blink every 1 second.

Once we build and flash, we should see a blinking LED on the breadboard.
