This project is a continuation of the previous blinky example. We will make some best practice improvements and continue to learn how Zephyr works under the hood to develop programs and applications in an RTOS.

# Improvements
The first order of business is to define a variable for pins. This helps with readability and refactoring. Zephyr's goal with the devicetree is to decouple the source code. This helps with abstracting away physical IO pins and target different boards without changing application code.

Zephyr provides a struct called `gpio_dt_spec` which contains members which fit the arguments `gpio_pin_configure` and other functions need. Instead of passing raw values into these functions, we can set up structs to pass information which produces a decoupled situation.

Of course we can't simply pass raw data into the `gpio_dt_spec` struct since that would defeat the whole purpose of decoupling. Instead, we can change the devicetree to include this information and have the program call this data from the devicetree at runtime.
# Devicetree Generation
A devicetree is made up of layers of files defining the devicetree and this structure differ between boards and architectures. There are 2 types of files used to define devicetrees:
- **Devicetree Sources (.dts, .dtsi)** are files which describe the actual hardware layout. These files will show you what hardware exists and how they're connected.
- **Devicetree Bindings (.yaml)** are files which describe what properties are valid for a given `compatible` string and how those properties map to Zephyr subsystems. An example if telling Zephyr's build system how to parse a specific node.
Devicetrees also have .overlay files which help extend nodes in devicetrees. We will use this for our purposes.

## Overlay File
```overlay
/ {
    leds {
        compatible = "gpio-leds";
        blinking_led: blinking_led {
            gpios = <&gpio0 25 GPIO_ACTIVE_HIGH>;
        };
    };
};
```
This file is created in the project root folder. `leds` will be our initial node just below the root node (`/`).  The `compatible` property will refer to the `.yaml` file will be using. This is provided by Zephyr. We then create a new node `blinking_led` with a node label of the same name and we define its gpio property. We specify the gpio0 device, pin number, and the flag. Note that the structure `<& specifier1 specifier2 ...>` is called a **phandle array** and is an array of pointers to other nodes in a devicetree.

> You'll need to make sure to build the application after this step so that the overlay file is applied and usable in code.

## Refactoring main.c
```c
static const struct gpio_dt_spec led = GPIO_DT_SPEC_GET(DT_NODELABEL(blinking_led), gpios);
```
We know need to make the changes in main.c that will decouple the program file. We start by changing the `device` struct into an `gpio_dt_spec` struct which will have the members we need.

```c
if (!device_is_ready(led.port)) {
	return;
}
```
In the code above, instead of passing the full struct, we now only pass in the port member into the function.

For the next section, we have two functions that can serve our purpose. The first is `gpio_pin_set_dt(&led, 1)` which will set the led to ON. However, since we are simply blinking the LED, we can also use `gpio_pin_toggle_dt(&led)` which simply toggles the current state of the pin. We'll use the latter for simplicity.
```c
while (true) {
	ret = gpio_pin_toggle_dt(&led);
	if (ret != 0) {
		return;
	}
	k_msleep(1000);
}
```
Since we ae toggling, we can remove the other section of code in our while loop.

After building I kept getting an error related to `blinking_led` label not being found in my device tree. Turns out the `.overlay` file has to exist in `helloworld/build` directory and the name should match the board name. Since Windows doesn't allow forward slash characters in file names, the correct name of the file should be `esp32_devkitc_esp32_procpu.overlay`. Once this was added to the `../board` directory, it built just fine.

# Summary
In this section, we refactored our Zephyr project to be depend on devicetree definitions rather than having raw pin access within the program itself.

The next session will focus on [[Zephyr Button Interrupts]]