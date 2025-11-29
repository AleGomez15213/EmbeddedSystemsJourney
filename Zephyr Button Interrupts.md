# Improvements
The purpose of this project is to continue iterating on our Zephyr blinky project. Our next step is to program the ability to turn the LED on and off manually using a button.
We connect our button. The button is connected to pin 13 of the ESP32.

## Overlay File
To start, we define a new device in our `.overlay` file. Under our root node, we add:
```
buttons {
	compatible = "gpio-keys";
	button: button {
		gpios = <&gpio0 13 GPIO_ACTIVE_HIGH>;
	};
};
```
We make use of `gpio-keys` which is a commonly used devicetree binding for buttons and keys type inputs. We then define our button to be connected to `gpio0` device at pin 13 and set it to `GPIO_ACTIVE_HIGH`. This is very similar to our previous LED definition.

## Main.c
In our main file, we define a new struct for our button.
```c
static const struct gpio_dt_spec button = GPIO_DT_SPEC_GET(DT_NODELABEL(button), gpios);
static struct gpio_callback button_cb_data;
```
The button struct is very similar to our LED struct. It is of type `gpio_dt_spec` and is set with the `GPIO_DT_SPEC_GET()` function. We pass in the node label, and the `gpios` property from the devicetree.

The second line is a GPIO interrupt callback struct. This is used to register what function should be called when the interrupt occurs. The struct stores:
- The callback function
- GPIO pin mask that triggers it
- Internal data used by GPIO driver

We will come back to this struct in a short bit.

```c
if (!device_is_ready(button.port)) {
	return 0;
}

...

ret = gpio_pin_configure_dt(&button, GPIO_INPUT | GPIO_PULL_UP);
if (ret != 0) {
	return 0;
}
```
Like the LED, we check if the device is ready and then configure the button. `gpio_pin_configure_dt()` takes in our button struct, and configuration flags. In this case, we are telling Zephyr to configure this button as an input (`GPIO_INPUT`) and to use the internal pull up (`GPIO_PULL_UP`). Because these flags are actually bitmasks, we are able to combine them to pass all desired flags at once.

Next, we call two config functions:
```c
    gpio_init_callback(&button_cb_data, button_pressed, BIT(button.pin));
    gpio_add_callback(button.port, &button_cb_data);
```

`gpio_init_callback()` initializes our button callback struct with the desired callback function (`button_pressed`), and the pin mask that triggers it.
	`BIT(button.pin)` converts the pin number to a bitmask so that the driver can use this mask to match the correct pin
`gpio_add_callback()` is what actually adds the callback to be used by `gpio0` device. It essentially tells Zephyr *"When this GPIO pin triggers an interrupt, call the function inside this callback structure."*

## Callback Function
Lastly, we add the function that will be called when an interrupt is triggered. We previously called it `button_pressed` so let's keep that. Above our main function, we write:
```c
void button_pressed(const struct device *dev, struct gpio_callback *cb, uint32_t pins) {
	int ret;
	ret = gpio_pin_toggle_dt(&led);
	if (ret != 0) {
		printk("Could not toggle LED\n");
	}
}
```
Very simply, the function takes in a `device` struct, a `gpio_callback` struct, and a pin represented by a `uint32_t` variable.

The function will simply toggle the LED state and if an error occurs, it will print a message to the system console. At this time, we won't see any messages printed because the console has not been configured.

## Optional
The tutorial we follow uses hardware debouncing with a 555 timer however, since we don't have those components, we will use software debouncing to prevent multiple rapid triggers when the button is pressed.

The basic idea is...
1) Disable interrupt temporarily
2) Start a timer using `k_timer` for debounce period
3) Re-enable interrupt after the timer expires

In our main.c file, we write...
```c
k_timer_init(&debounce_timer, debounce_timer_handler, NULL);
```
which takes in a `k_timer` struct and a handler function called `debounce_timer_handler()`. The final parameter is the stop handler (What function should be called when the timer is stopped manually). In this case, we have no use for this so we pass in `NULL`.

We then add...
```c
void debounce_timer_handler(struct k_timer *dummy)
{
    // Re-enable button interrupt after debounce time
	gpio_pin_interrupt_configure_dt(&button, GPIO_INT_EDGE_TO_ACTIVE);
}
```
and declare `static struct k_timer debounce_timer` at the top of our file.

Lastly, we add the line
```c
k_timer_start(&debounce_timer, K_MSEC(50), K_NO_WAIT)
```
in our callback function so that the timer begins upon pressing the button.

Moving on, we will set up [[PWM and Timers in Zephyr]]. Soon, we can begin actual development on the smartwatch.