`uint32_t` is a type alias defined in the C standard library `<stdint.h>`. It represents an unsigned integer of exactly 32 bits. The variable can save integer values ranging from `0` to `2^32 - 1`.
- Max number is `2^32 - 1` because this is an unsigned value. All bits are used for magnitude not sign and `- 1` comes from the fact that we start counting from 0 not 1.

# Comparison to `int`
Unlike a regular `int` variable, `uint32_t` is always 32 bits regardless of the platform. This is useful when you need exact bit-width control.

Examples include:
- **Accessing hardware registers:** Many peripheral registers (GPIO, timers, UART, etc.) have an exact bit size. If we set them to a number greater than their size, we could write the wrong number of bits, overwrite other memory, or trigger undefined behavior.
	- **Memory-Mapped I/O:** Some sensors use memory-mapped registers with specified widths. In the example below, using `int` or `short` might give a different result on different compilers or architectures.
```c
volatile uint16_t *temp_sensor_reg = (uint16_t *)0x50000010;
uint16_t temperature = *temp_sensor_reg;
```