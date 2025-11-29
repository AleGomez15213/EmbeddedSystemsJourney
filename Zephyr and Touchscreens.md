How complex this section is will ultimately be determined by what LCD screen you buy. I happened to purchase this [Waveshare 1.28in LCD screen](https://www.waveshare.com/wiki/1.28inch_Touch_LCD) that happened to use the GC9A01 driver. This driver is supported out-of-the-box in Zephyr and can be found under **"..\zephyr\drivers\display\display_gc9x01x.c."**

# Overlay File
We need to tell Zephyr how we want to configure our LCD screen. This overlay file below does 3 things:
1. Designates LCD as system's default display
2. Creates a MIPI-DBI wrapper
3. Define and configure SPI2

```dts
#include <zephyr/dt-bindings/display/panel.h>
#include <zephyr/dt-bindings/mipi_dbi/mipi_dbi.h>
```
The headers above will bring in constants that will be used throughout the file. These constants include pixel formats (RGB565, RGB888, etc.), MIPI_DBI modes, and frequency macros.

```dts
/ {
    chosen {
        zephyr,display = &gc9x01x_lcd;
    };
```
This is the top level node of the tree. It will set the default display. This ensures that LVGL will automatically bind to the device. Later one when we main `main.c`, we will use `display_get_default()` which returns this node.

```dts
    mipi_dbi0: mipi_dbi0 {
        compatible = "zephyr,mipi-dbi-spi";
        status = "okay";
```
The next step is to create the MIPI-DBIP wrapper. We create a virtual device `mipi_dibi0` that will speak over SPI. This is important because the driver we will use `GC9A01` uses DBI operations, not raw SPI transactions
...
# Debugging
Eventually, I got backlight to turn on however nothing was being drawn to the screen. I first verified that my file was working by using some print messages and monitoring the chip using `west espressif monitor`. The app was able to run without problems and even made it to the line that actually draws something to the screen however it was still blank.

After ensuring my `main.c` file was not the issue, I verified my GPIO wiring. All wires seem to match what was established in my overlay file as well as my `zephyr.dts` file found under the build folder. For a while I was stumped and use ChatGPT to automate a lot of the file comparisons but did not find anything. Eventually however, I check the `spim2_default` node in my `zephyr.dts` file and found some important notes.

![[Pasted image 20251117131305.png]]
The snippet above shows some notes about where the `spim2_default` node is defined in a pinctrl file. I went to line 44 as directed and discovered that `spim2_default` had different GPIO pins defined for the SPI wires.

> Found in zephyrproject\zephyr\boards\espressif\esp32_devkitc\esp32_devkitc-pinctrl.dtsi
```dts
spim2_default: spim2_default {
	group1 {
		pinmux = <SPIM2_MISO_GPIO12>,
		<SPIM2_SCLK_GPIO14>,
		<SPIM2_CSEL_GPIO15>;
	};
	group2 {
		pinmux = <SPIM2_MOSI_GPIO13>;
		output-low;
	};
};
```

After adjusting my overlay file to match these pins (GPIO14 is SCLK, GPIO15 is LCD_CS, GPIO2 is LCD_DS), I ran the build command again and flashed the ESP32.

## Flash Error
![[Pasted image 20251127130951.png]]
After flashing, I got this error (likely a result of my rewiring.) After investigating a little bit, I discovered the reason why my previous flashing worked, and this one didn't.

ESP tool can't talk to the SPI flash at all so it can't figure out the flash size. This is happening because I connected a wire to the ESP's strap pins (GPIO12 and GPIO15) and this can interfere with the ability to flash the ESP32.
	This can also be tested by unplugging the LCD screen and flashing again. Flashing this works just fine meaning the LCD screen wiring is interfering with the ability to flash.

## ESP32 Strap Pins
SP32 strapping pins are GPIO pins that are sampled by the chip during startup to determine the initial boot mode and configuration. They are used to set parameters like entering bootloader or flashing mode, and misconfiguring them can prevent the chip from starting correctly. On our version, the strapping pins include:
- GPIO12 which controls flash voltage
- GPIO0, 2, 4, 5, 15 which determine boot mode, etc.

One solution is to unplug the LCD screen when flashing. This ensures the LCD startup functions do not interfere with the ESP flash. The better solution (at least until we run out of available GPIO pins) is to change the wiring.

We will rewire following the guide below:
- **SCLK → GPIO18**
- **MOSI → GPIO23**
- **CS → GPIO5**
- **DC → GPIO21**
- **RESET → GPIO22**

Then override the SPI2 pinctrl and cs-gpios in my overlay so Zephyr can actually use them.

## Color Selection
![[Pasted image 20251127133508.png]]
Looks like the wiring works and the screen is painted. The only problem is the code is supposed to paint the screen red and this is very clearly not red.

