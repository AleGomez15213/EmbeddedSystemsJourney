SPI is a 4 wire serial interface and is usually faster than UART or the [[I2C Protocal]]. It is usually used to transfer data from a "smart" controller to a "less smart" peripheral device. The protocol is usually used by sensors, displays, game controllers, ADC/DACs, LCD screens, etc.

# Wires
SPI always has one master and one or more slaves. Three out of 4 wires can be shared in a bus configuration among the slaves but each slave must have it's own **chip select** wire to differentiate which slave will receive/send data.
## Chip Select (CS)
CS is active low and is indicated by an overbar. When the line is pulled low, the chip is selected for communication and the slave knows to listen for a clock signal as well as data. The line is kept low until the communication is complete.
- This method of selecting different targets is simpler than I2C in that it does not require the message to define a specific slave address. Instead, the line is simply pulled low for the desired target.
## SCLK
## MOSI
## MISO