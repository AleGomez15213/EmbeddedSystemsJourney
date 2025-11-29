Using `printk()` is very useful for debugging purposes. In this section, we'll update our project to enable the system console and get `printk()` working.

We start of by updating our `prj.conf` file:
```ini
CONFIG_PRINTK=y
CONFIG_CONSOLE=y
CONFIG_UART_CONSOLE=y
```

We also need to ensure that the `zephyr.dts` file contains definitions for UART devices. In our case, it already does.
```
uart0: uart@3ff40000 {
	compatible = "espressif,esp32-uart";
	reg = < 0x3ff40000 0x400 >;
	...
	status = "okay";
	current-speed = < 0x1c200 >;
};
```
And we verify that the chosen node in `zephyr.dts` also includes the following:
```
/ {
    chosen {
        zephyr,console = &uart0;
        zephyr,serial = &uart0;
        zephyr,stdout = &uart0;
    };
};
```
If not present, it can be defined in a `.overlay` file.

We then use a program like PuTTY to access the console:
1) Plug in ESP32 into USB port. This will automatically expose a **COM port**
2) Open Device Manager (Windows) to find the COM port number (ex: COM5)
3) Open PuTTY and configure settings:
	- Connection type: Serial
	- Serial Line: COM5
	- Speed: 115200 (Or whatever is listed under `current-speed` in the zephyr dts file. Sometimes, it will be listed in hex format.)
Now open PuTTY session and any printed messages should show in the terminal.
	**Note:** You cannot flash and have a PuTTY session at the same time. You will get an error saying the port is busy. Instead, flash the ESP and then open the PuTTY session. Opening the session will restart the ESP.