Inter-Integrated Circuit or I2C is a synchronous master-slave protocl. It uses `SDA` (Serial Data) and `SCL` (Serial Clock) wires to communicate. Multiple slaves can be connected to one master and are differentiated by their **I2C address**.

# Frames
I2C communication is organized into a frame. Each frame is split into different sectionsbeginning  with **start** which is denoted by the SDA line going low followed by the SCL line going low. The next section is the **slaved address**, then comes the **read/write** which indicates whether the communication wants to read or write data to the slave. The next section is the **acknowledge** bit or ACK, followed by the **data**, another ACK, and lastly the **stop** frame.
![[Pasted image 20250628130302.png]]
## Start
In idle state, both `SDA` and `SCL` are high. The start condition occurs when `SDA` is pulled low, followed by `SCL` being pulled low. This action "claims the bus." The node that claims the bus is considered the master and prevents other nodes from taking control of the bus.

Once the bus is claimed, the master node will begin to send a clock signal. 
## Slave Address
Each I2C device must have a unique address. On most devices, these addresses are partially configurable.
- The address is 7 bits long, MSB first
- 10 bit addresses are also used but uncommon
## Read/Write
Usually 1 bit long. This indicates if the incoming message will be a read operation or a write operation on the slave.
- 0 -> master writes to slave
- 1 -> master reads from slave
## Acknowledge
The acknowledge bit is sent by a receiver after every byte is sent.
- 0 -> acknowledgement (ACK)
- 1 -> negative acknowledgement (NACK)
If the receiver sends a NACK, the sender knows to stop or retry, otherwise, it will continue sending data.
	`ACK` after a slave address confirms that it is active on the bus and is ready to read/write.
	`ACK` after data is confirming that the data was properly received
## Data
Data byte contains the actual information being transferred. It can include memory or register content, addresses, etc. **Many data bytes can be sent within one frame.**
- Always 8 bits long, MSB first.
- Always followed by an acknowledgement bit which is set to 0 if the message was received properly.
## Stop
Similar to the start condition, a stop condition indicates the end of an I2C message frame. The condition follows this patter:
- SCL returns (and remains) high
- SDA returns (and remains)* high
Since SDA only transitions when SCL is low, anything else would indicate a stop condition. Once this happens, the clock signal stops and any node can "claim the bus" in order to begin new communication.
# SDA and SCL relationship
`SDA` does not change between clock rising and falling edges. In other words: *During data transmission, `SDA` only changes when `SCL` is low*. This is important because, as mentioned previously, an SDA transition when SCL is **high** indicates a start or stop condition.
![[Pasted image 20250628131005.png]]

# Pull Up Resistors
Pull up resistors are used in order to ensure that the I2C line is high in the idle state. Sometimes, I2C is called an "open drain" system for this reason. Each I2C device contains logic that can open and close the drain to connect the line to GND or VCC.

Pulling down a line is usually **much faster** than pulling the line up. This time is dependant on bus capacitance and values of the pull up resistors.
![[Pasted image 20250628133241.png]]
- **Higher resistance** increase pull up time and **limit bus speed.**
- **Lower resistance** decreases pull up time (faster communication) but **requires higher power.**
- Values can range from 1kOhm to 10kOhms.
## Modes (Bus Speeds)
![[Pasted image 20250628133453.png]]