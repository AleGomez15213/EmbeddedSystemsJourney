A couple of important words:
- **Pulse Width** The distance from the beginning of a pulse to the end of a pulse.
- **Period** is the distance between the end of one pulse to the end of the next pulse.
- **Duty Cycle** is calculated by the following equation:
$$
Duty\space Cycle = \frac{Pulse\space Width}{Period}
$$
	This is the proportion of the period that the pulse lasts. When attempting to use PWM to mimic analog voltage, the duty cycle would describe how much voltage a node has. It ranges from 0%-100%.

# Improvements
To begin programming PWM into our helloworld application, we'll need to update the overlay file and find the appropriate `compatible` property. We can go to https://docs.zephyrproject.org/latest/build/dts/api/bindings.html which lists all the bindings Zephyr has. We find a PWM binding listed under **Generic or vendor-independent > pwm-leds**. We see that the binding has one required property: *pwms* which is a reference to a PWM instance. Because this property is of type *phandle-array* (which takes in a device and specifier cells) we'll likely need to reference some kind of controller with some relating parameters.

# Finding Controllers
We can find available controllers for the ESP32 here: https://docs.zephyrproject.org/latest/build/dts/api/bindings.html#dt-vendor-espressif. If we don't find anything appropriate here, we can also look through the `.yaml` files in the filesystem.

In our case, we look through `..\zephyrproject\zephyr\dts\bindings` to find a led pwm related binding. In addition to the motor controller pwm file we find online, we also see a file called `espressif,esp32-ledc.yaml`. This looks promising.

Upon inspection of the file, we see some important details:
```yaml
compatible: "espressif,esp32-ledc"

include: [pwm-controller.yaml, pinctrl-device.yaml, base.yaml]
```
We have the compatible which we'll be referencing in our `.overlay` file. We also see the file includes three other yaml files with each one requiring its own properties to be specified. Note that `pwm-controller.yaml` requires 3 specifiers to be provided in order to use. This refers to the pwms (what we use when passing in arguments in our overlay file) property. We also see this reflected in the next line below:
```yaml
properties:
	"#pwm-cells":
		const: 3
```
- Note: Although pwm-cells and pwms do not match, Zephyr documentation specifies that a trialing 's' is removed when refering to properties so although the names do not match entirely, this is the correct property. 

## Overlay File
With this newfound information, we can write our overlay file:
```
/ {
    pwmleds {
        compatible = "pwm-leds";
        fading_led:fading_led {
            pwms = <&ledc0 0  10000 PWM_POLARITY_NORMAL>;
        };
    };
};
```
We specify the compatible we want to use. We then create a child node called `fading-leds` with a phandle-array property `<&ledc0 0  10000 PWM_POLARITY_NORMAL>`. This specifies the controller `ledc0`, channel 0, period of 10kHz, and normal polarity or active high.

# Main.c
Let's begin coding here:
```c
#include <zephyr/kernel.h>
#include <zephyr/drivers/pwm.h>
#include <zephyr/device.h>

static const struct pwm_dt_spec fading_led  = PWM_DT_SPEC_GET(DT_NODE(fading_led));
```

## Errors
If we were to build the application at this point, we would get a cryptic error talking about not declaring `__device_dts_ord_28`. One way to troubleshoot is to visit the Zephyr troubleshooting guide.

Here we find that the build process will throw away many of the intermediary files after they're no longer needed but we can keep them by using the `--DEXTRA_CFLAGS=-save-temps=obj` flag. While we still get the same error, we can now search for `__device_dts_ord_28` and get a reference in the `main.c.i` file. As it turns out, in the `zephyr.dts` devicetree, `ledc0` has a status property that says it is currently disabled. Therefore, we can update our overlay file to enable this controller.
```
/ {
    pwmleds {
        ...
    };
};

&ledc0 {
	status = "okay";
}
```

After adding this, we build and it works. We can continue coding:
```c
int main(void) {
	if (!device_is_ready(fading_led.dev)) {
		printk("Error: PWM device %s is not ready\n", fading_led.dev->name);
		return 0;
	}
}
```
This is pretty simple, it checks if the device is ready before it can be used. If we build now however, we get another error similar to the previous one. It complains about the same object however this time, it says that `__device_dts_ord_28` is undefined. This is a linker error. For this, we look at where `PWM_DT_SPEC_GET()`is defined.

Turns out `PWM_DT_SPEC_GET` uses another macro which uses another macro, etc. and eventually we get to `DEVICE_DT_GET` which has this comment:

```
If no such device was allocated, this will fail at linker time. If you get an
 * error that looks like `undefined reference to __device_dts_ord_<N>`, that is
 * what happened. Check to make sure your device driver is being compiled,
 * usually by enabling the Kconfig options it requires.
```

## Using Kconfig
We can use https://docs.zephyrproject.org/latest/kconfig.html to check different Kconfig options by name. Although `CONFIG_LED_PWM` and `CONFIG_PWM_LED_ESP32` are both turned on by default, the config option they depend on, `CONFIG_PWM` is not turned on by default. Therefore we must turn it on.

This seems to work.

## Using pinctrl
One last thing to do before we continue. We need to tell our system which pin will be using PWM for the LED. Since we are using the `ledc0_default` we want to use something called pinctrl.

Note that Pin control is different thant the GPIO drivers we previously used.
- **Pin Control** is meant to be handle pin configuration and multiplexing for peripherals (UART TX, I2C SDA, etc,)
- **GPIO Dirvers** are used to set pins as input or output, or when you want to detect interrupts.

Therefore, we want to use pin control for PWM. To do so, we add to the overlay file
```
&pinctrl {
    ledc0_default: ledc0_default {
        group1 {
            pinmux = <LEDC_CH0_GPIO25>;
            output-enable;
        };
    };
};

&ledc0 {
    pinctrl-0 = <&ledc0_default>;
    pinctrl-names = "default";
	status = "okay";
    #address-cells = <1>;
    #size-cells = <0>;
    channel0@0 {
        reg = <0x0>;
        timer = <0>;
    };
};
```
We add a new node `pinctrl` which has a child node `ledc0_default`. We define group1 with the desired pinmux option and make sure we include the `output-enable` flag.
## Implementing PWM
To add PWM, we'll need to set the LED's voltage level to be different and increase/decrease it rapidly. Additionally, we'll need some kind of way to keep track of time. Instead of using the `ksleep()` function, we can use Timers.

```c
#define NUM_STEPS 100U
#define SLEEP_DELTA_MSEC 20U

static const struct pwm_dt_spec fading_led  = PWM_DT_SPEC_GET(DT_NODELABEL(fading_led));

static uint32_t pulse_width_nsec = 0U;
static uint32_t pulse_width_delta_nsec = 0U;
static uint32_t steps_taken = 0U;
static bool increasing_intensity = true;

int ret;
```
- `NUM_STEPS` gives the total number of steps to be taken by the PWM. It subdivides the LED's brightness meaning that a greater number will appear like a more smooth transition between dark and bright.
- `SLEEP_DELTA_MSEC` defines how frequently the timer callback runs.
- `pulse_width_nsec` keeps track of the time that the LED is active. It controls how bright the LED appears.
- 

```c
void led_delta_timer_handler(struct k_timer *timer_info) {
    if (increasing_intensity) {
        if (steps_taken < NUM_STEPS) {
            ret = pwm_set_pulse_dt(&fading_led, pulse_width_nsec);
            steps_taken++;
            pulse_width_nsec += pulse_width_delta_nsec;
        } else {
            increasing_intensity = false;
            steps_taken--;
            pulse_width_nsec -= pulse_width_delta_nsec;
        }
    } else {
        if (steps_taken > 0) {
            ret = pwm_set_pulse_dt(&fading_led, pulse_width_nsec);
            steps_taken--;
            pulse_width_nsec -= pulse_width_delta_nsec;
        } else {
            increasing_intensity = true;
            steps_taken++;
            pulse_width_nsec += pulse_width_delta_nsec;
        }
    }
}
```
The function `led_delta_timer_handler` is the timer callback function and will run every time the timer executes. The system will first check if the LED is increasing intensity or decreasing intensity.
If the brightness isn't at max yet, the current brightness is set by `pwm_set_pulse_dt()` which takes in the LED and the PWM value.

The program then updates `steps_taken` to keep track of how many steps have been taken and the duty cycle is increased by `pulse_width_delta_nsec`. If the max amount of steps is reached, we switch the `increasing_intensity` bool to false (indicating we will begin deceasing brightness), `steps_taken` is reduced, and the duty cycle is decreased.

The next time the timer goes off, the `else` block is ran and everything is done in reverse to reduce brightness.

In the next video, we learn how to make a [[Custom Zephyr Driver]]