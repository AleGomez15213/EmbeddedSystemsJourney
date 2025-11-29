Like the title implies, this project will seek to learn how to write custom drivers for Zephyr. In this case, we'll write a driver to control a simple stepper motor then set it up as a **Zephyr device instance** in order to reuse the code.

# Writing Stepper Motor Driver
We will first hard-code everything in the `main.c` file and later improve the code.

As a first step, we will write a function that can be called to make the motor "take a step." We start off by including any header files and variables we may need.
```c
#include <stdint.h>
#include <zephyr/device.h>
#include <zephyr/drivers/gpio.h>
#include <zephyr/logging/log.h>

LOG_MODULE_REGISTER(steper_driver, LOG_LEVEL_DBG);
static const gpio_pin_t IN[] = {25, 26, 27 14};
static const struct device * gpio_dev = DEVICE_DT_GET(DT_NODELABEL(gpio0));
```
The `stdint.h` header file contains definitions for `uint32_t` data types. `log.h` will be used when we set up logging. We also use a macro `LOG_MODULE_REGISTER` to set up the logger. Lastly we set up the pins as an array called `IN` and the device `gpio_dev` similar to how we set up devices previously.

```c
enum rotation_direction {
    CLOCKWISE,
    COUNTER_CLOCKWISE
};
```
We also create an enum with two options: cloclwise or counter-clockwise to denote direction of our stepper motor.

```c
int take_steps(const uint32_t target_num_steps, const enum rotation_direction rot_dir, const int32_t sleep_time_ms) {
    if (!device_is_ready(gpio_dev)) {
        LOG_ERR("device %s is not ready", gpio_dev->name);
        return -ENODEV;
    }
    return 0;
}
```
The `take_steps` function will return a status code as an integer. It takes in the number of steps requested, direction, and a sleep_time parameter. It first checks if a device is ready, if it's not, log the error. Otherwise, continue through the function.

```c
for (int i = 0; i < sizeof(IN); i++) {
	if (gpio_pin_configure(gpio_dev, IN[i], GPIO_OUTPUT_LOW != 0)) {
		LOG_ERR("could not set gpio pin %s to low", IN[i]);
		return -EIO;
	}
}
```
Now we turn off all gpio pins for motor by setting them to low.

```c
uint32_t curr_steps_count = 0U;
while (curr_steps_count < target_num_steps) {
	k_msleep(sleep_time_ms);
	gpio_pin_set_raw(gpio_dev, IN[3], 0);
	gpio_pin_set_raw(gpio_dev, IN[1], 1);
	curr_steps_count++;

	k_msleep(sleep_time_ms);
	gpio_pin_set_raw(gpio_dev, IN[0], 0);
	gpio_pin_set_raw(gpio_dev, IN[2], 1);
	curr_steps_count++;

	k_msleep(sleep_time_ms);
	gpio_pin_set_raw(gpio_dev, IN[1], 0);
	gpio_pin_set_raw(gpio_dev, IN[3], 1);
	curr_steps_count++;

	k_msleep(sleep_time_ms);
	gpio_pin_set_raw(gpio_dev, IN[2], 0);
	gpio_pin_set_raw(gpio_dev, IN[0], 1);
	curr_steps_count++;
}
```
Lastly, we (somewhat crudely) change the pins appropriately so as to make the motor complete a revolution. This is done until the `curr_steps_count` exceeds the `target_num_steps`. Note that the `target_num_steps` must be divisible by 4 for this to work appropriately. For example, a value of 2048 would complete a single revolution.

We then adjust this code to take into account `CLOCKWISE` selection and `COUNTER_CLOCKWISE` selection, and call the function in the main loop.