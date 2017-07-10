---
title: Socket API over ESP
breadcrumbs:
 - name: 'Hardware'
   url: '/hardware'
versions:
 - name: 'Rick Main Firmware'
   value: 'v.1.0.0'
 - name: 'Rick ESP Firmware'
   value: 'v.1.0.0'
 - name: 'Rick Control Poard PCB'
   value: 'v.1.0.0'
layout: article
---

<br>

The `ESP8266` WiFi modem on Rick presents a low-level TCP Socket and Web Socket API to the network.
Clients connecting to this API can send byte-level commands to the ESP which will in turn instruct
the main control chip to perform an action or report back on a sensor. The socket API runs
over port `24`. It's also accessible over Serial or i2c, in ROSSerial format with topicID corresponding to socket_cmd (112)
{:.feature}

There's a *Test Harness* available that can connect to Marty and just print out a stream
of Sensor data. This uses JavaScript and a Web Socket connection. Incidentally, this is
how Scratch talks to Marty.

<br>

<div class="center">
    <a href="/hardware/esp-socket-harness" class="btn rounded">
        <i class="fa fa-fw fa-wifi muted"></i> &nbsp; Test Harness for the Socket API
    </a>
    <a href="/hardware/esp-socket-discovery" class="btn rounded">
        <i class="fa fa-fw fa-search muted"></i> &nbsp; Simple Discovery Tool
    </a>
</div>


Ports & Services
===


| Number    | Use                                                             |
|-----------|-----------------------------------------------------------------|
| `23`      | Unmanaged sockets                                               |
| `24`      | Managed sockets                                                 |
| `80`      | "Marty Setup" functionality, JavaScript-based service discovery |
| `81`      | Websockets                                                      |
| `4000`    | UDP-based service discovery                                     |


*Managed* sockets follow the below socket API, while *Unmanaged* sockets are given raw
pass-through access to the Serial line between the ESP and the main STM446 microcontroller.

In almost all use-cases, the managed socket is what you'll want to use.

<br>

------------------------------------------------------------------------------------------------


Socket API
===

Over Ports 23, 24 & 81
---
{:.subhead}


`GET` Type Packets
---

These `GET` requests fetch sensory information. A `GET` request is characterised by the zeroth
packet byte being `0x01`. The 2nd byte then selects the sensor type, and the third selects
the ID, if applicable. Where the Type is undefined, any value may be sent (e.g. `0x00`) but
it must be there so the packet length is correct. All `GET` packets are of length 3.

| Byte 0 | Byte 1  | Byte 2  |
|--------|---------|---------|
| 0x01   | *Type*  | *ID*    |
{:.bitfield}

Currently there are 4 types of sensor queryable: Battery voltage, Accelerometer, Motor Currents and GPIOs.


| Sensor           | Type Byte  | ID Byte           | ID Description      | Return type |
|:-----------------|:-----------|:------------------|:--------------------|:------------|
| *Battery*        | 0x01       | *(undefined)*     | *Not applicable*    | float32     |
| *Accelerometer*  | 0x02       | [0x00, ..., 0x02] | x, y and z axes     | float32     |
| *Motor Current*  | 0x03       | [0x00, ..., 0x07] | Motors 0 through 7  | float32     |
| *GPIO*           | 0x04       | [0x00, ..., 0x07] | GPIOs 0 through 7   | float32     |
| *Chatter*        | 0x05       | *(undefined)*     | *Not applicable*    | int32 length, string |
| *Motor Position* | 0x06       | [0x00, ..., 0x08] | Motors 0 through 8  | int8        |
| *Motor Enabled*  | 0x07       | [0x00, ..., 0x08] | Motors 0 through 8  | bool        |
{:.tt}

Battery, Accelerometer, Motor current, and GPIO readings returned after a `GET` request are a 4 Byte Little-Endian float, i.e. the
LSB is first, the MSB last.

Chatter returns a packet length as a 4-byte Little-Endian int, then a null terminated string.

Motor Position returns an int8, in the range -100 to +100.

Motor enabled returns a bool.

The 8th motor channel (usually the eyes on Marty) does *not* have a current sensor
so no value can be reported.

<br>


`COMMAND` Type Packets
---

Command packets intstruct the control board to do something. Similar to the `GET` type packets,
`COMMAND` packets all begin `0x02`. This type of packet can vary
in length depending on the operation, so the 1st and 2nd bytes encode the payload size
(number of *bytes*), though don't count themselves nor the zeroth byte in the size.

| Byte 0 | Byte 1     | Byte 2     | Byte 3           | Byte 4    | ... | Byte M    |
|--------|------------|------------|------------------|-----------|-----|-----------|
| 0x02   | *Size LSB* | *Size MSB* | *Command Opcode* | *1st Arg* | ... | *Nth Arg* |
{:.bitfield}

The payload size (1st and 2nd bytes) should be encoded as a little endian integer.
Bytes in position 4 through M depend on the opcode and command being called.

Here is a summary table of the commands, more detail on specific functions is below.

| Command                | Size | Opcode   | Arguments                                               |
|:-----------------------|:-----| :--------|:--------------------------------------------------------|
| [*hello*](#cmdHello)   | 1 (2)| 0x00     | [uint8 type *(default 0)*]                              |
| [*lean*](#cmdLean)     | 5    | 0x02     | uint8 direction, int8 amount, uint16 move\_time         |
| [*walk*](#cmdWalk)     | 7    | 0x03     | uint8 steps, uint8 turn, uint16 move\_time, <br />int8 step\_length, int8 side |
| [*kick*](#cmdKick)     | 5    | 0x05     | uint8 side, int8 twist, uint16 move\_time               |
| [*celebrate*](#cmdCelebrate)            | 3    | 0x08     | uint16 move\_time                      |
| [*tap\_foot*](#cmdTapFoot)            | 2    | 0x0A     | int8 side                                |
| [*arms*](#cmdArms)     | 5    | 0x0B     | int8 r\_angle, int8 l\_angle, uint16 move\_time         |
| [*sidestep*](#cmdSidestep)             | 6    | 0x0E     | int8 side, int8 num\_steps, uint16 move\_time, int8 step\_length  |
| [*stand straight*](#cmdStandStraight)       | 3    | 0x0F     | uint16 move\_time                  |
| [*play\_sound*](#cmdPlaySound)          | 7    | 0x10     | uint16 freq\_start, uint16 freq\_end, uint16 duration   |
| [*stop*](#cmdStop)                 | 2    | 0x11     | uint8 stop\_type (0x00, ..., 0x05)          |
| [*move\_joint*](#cmdMoveJoint)          | 5    | 0x12     | uint8 joint\_id, int8 position, uint16 move\_time       |
| [*enable\_motors*](#cmdEnableMotors)       | 4    | 0x13     | [uint16_t motorFlags, [int8_t mode (0x00, 0x01)]]    |
| [*disable\_motors*](#cmdDisableMotors)      | 4    | 0x14     | [uint16_t motorFlags, [int8_t mode (0x00, 0x01)]]   |
| [*fall\_protection*](#cmdFallProtection)     | 2    | 0x15     | bool enabled (0x00 or 0x01)       |
| [*motor\_protection*](#cmdMotorProtection)    | 2    | 0x16     | bool enabled (0x00 or 0x01)      |
| [*low\_battery\_cutoff*](#cmdLowBattery) | 2    | 0x17     | bool enabled (0x00 or 0x01)           |
| [*buzz\_prevention*](#cmdBuzzPrevention)     | 2    | 0x18     | bool enabled (0x00 or 0x01)       |
| [*set\_IO\_type*](#cmdSetIOType)        | 3    | 0x19     | uint8 io_number, int8 type             |
| [*IO\_write*](#cmdIOWrite)            | 6    | 0x1A     | uint8_t io_number, float value           |
| [*i2c\_write*](#cmdI2CWrite)           | 1+*n*| 0x1B     | uint8[*n*] address + data               |
| [*circle\_dance*](#cmdCircleDance)        | 4    | 0x1C     | uint8 side, uint16 move_time         |
| [*lifelike\_behaviours*](#cmdLifelike) | 2    | 0x1D     | bool enabled (0x00 or 0x01)             |
| [*enable\_safeties*](#cmdEnableSafeties)     | 1    | 0x1E     | *-none-*                          |
| [*set\_parameter*](#cmdSetParameter)       | 2+*n*| 0x1F     | uint8 paramID, *params*             |
| [*get\_firmware\_version*](#cmdGetFirmwareVersion)|1    | 0x20     | *-none-*                      |
| [*mute\_esp\_serial*](#cmdMuteESPSerial)    | 1    | 0x21     | *-none-*                           |
| [*clear\_calibration*](#cmdClearCalibration)   | 1    | 0xFE     | *-none-*                        |
| [*save\_calibration*](#cmdSaveCalibration)    | 1    | 0xFF     | *-none-*                         |
{:.tt}

<br>

### <a name="cmdDirectionCodes"></a> direction / side codes

| Code |          |
|:-----|:---------|
| 0x00 | Left     |
| 0x01 | Right    |
| 0x02 | Forward  |
| 0x03 | Backward |
| 0x04 | *Any*    |
{:.tt}

### <a name="cmdMoveTime"></a>move_time

Move time specifies the number of milliseconds for a movement to take, and is always a uint16 with LSB first. So, to take 1 second, or 1,000 ms, send bytes [0xE8, 0x03].

#### <a name="cmdHello">`hello`</a>
*`uint8 type (default 0)`*

| Type    |    |
|:--------|:---|
| 0       | Default, move joints to zero position and wiggle eyebrows |
| 1       | Try to force enable. If Marty is out of position, this will try to<br /> ensure that Marty ends up centred with motors enabled. |
{:.tt}

#### <a name="cmdLean"></a>`lean`
*[`uint8 direction`](#cmdDirectionCodes)`, int8 amount, `[`uint16 move_time`](#cmdMoveTime)*

`amount` is an int specifying percentage of max, so 0-100.

#### <a name="cmdWalk"></a>`walk`
*`uint8 steps, int8 turn, `[`uint16 move_time`](#cmdMoveTime)`, int8 step_length, `[`int8 side`](#cmdDirectionCodes)*

`steps` is a uint8 specifying the number of steps <br />
`turn` specifies the amount to turn per step as a percentage, so -100 to 100<br />
`step_length` specifies the step length as a percentage of max, so -100 to 100.  Negative step lengths will walk backwards.<br />
`move_time` and `side` are as defined above

#### <a name="cmdKick"></a>`kick`
*[`uint8 side`](#cmdDirectionCodes)`, int8 twist, `[`uint16 move_time`](#cmdMoveTime)*

`twist` is the amount to twist the knee joint while kicking, from -100 to +100. It can be used to kick at an angle.

#### <a name="cmdCelebrate"></a>`celebrate`
*[`uint16 move_time`](#cmdMoveTime)*

Celebrate makes Marty do a dance

#### <a name="cmdTapFoot"></a>`tap_foot`
*[`int8 side`](#cmdDirectionCodes)*

Tap_foot will make Marty tap one of his feet three times

#### <a name="cmdArms"></a>`arms`
*`int8 r_angle, int8 l_angle, `[`uint16 move_time`](#cmdMoveTime)*

`r_angle` is the right arm position, from -100 to +100
`l_angle` is the left arm position, from -100 to +100

#### <a name="cmdSidestep"></a>`sidestep`
*[`int8 side`](#cmdDirectionCodes)`, uint8 num_steps, `[`uint16 move_time`](#cmdMoveTime)`, int8 step_length`*

`num_steps` is a uint8 specifying the number of steps
`step_length` is an int8 specifying the step length, as a percentage of max, so 0 to 100

#### <a name="cmdStandStraight"></a>`stand_straight`
*[`uint16 move_time`](#cmdMoveTime)*

Stand_straight returns all motors to the zero positions

#### <a name="cmdPlaySound"></a>`play_sound`
*`uint16 freq_start, uint16 freq_end, `[`uint16 duration`](#cmdMoveTime)*

play_sound will make the activate the buzzer on Marty, it'll start at `freq_start` and finish at `freq_end` after `duration` milliseconds. These sounds will queue on Marty, so multiple `play_sound` commands can be used to queue a tune.

`freq_start` is a uint16, LSB first, specifying the starting frequency in Hz
`freq_end` is a uint16, LSB first, specifying the ending frequency in Hz

#### <a name="cmdStop"></a>`stop`
*`uint8 stop_type`*

| Code | Stop type                |
|:-----|:-------------------------|
| 0    | Clear movement queue only (so finish the current step/wiggle/movement) |
| 1    | Clear movement queue and servo queues (freeze where you are)           |
| 2    | Clear everything and disable motors                                    |
| 3    | Clear everything, and make robot return to zero                        |
| 4    | Pause, but keep servo and movequeue intact and motors enabled          |
| 5    | As 4, but disable motors too                                           |
{:.tt}

#### <a name="cmdMoveJoint"></a>`move_joint`
*`uint8 joint_id, int8 position, `[`uint16 move_time`](#cmdMoveTime)*

`joint_id` the specific joint to move
`position` an int8 from -100 to +100 as a percentage of maximum servo movement. Note that not all motors can physically move through the whole range in Marty

| Joint ID | Joint in Marty             |
|:---------|:---------------------------|
| 0        | Left hip                   |
| 1        | Left twist                 |
| 2        | Left knee                  |
| 3        | Right hip                  |
| 4        | Right twist                |
| 5        | Right knee                 |
| 6        | Left arm                   |
| 7        | Right arm                  |
| 8        | Eyes                       |
{:.tt}


#### <a name="cmdEnableMotors"></a>`enable_motors`
*`[uint16_t motor_flags, [int8_t mode (0x00, 0x01)]]`*

enable_motors can be called with optional paramters. It will allow motors to be given positions, but won't by itself move the motors anywhere.<br/>#
It will also unpause movement.

`motor_flags` is a uint16 specifying which motors should be enabled. Byte 0 (the LSB) corresponds to motor 0, so to enable motors 0 and 5, you would send b0000 0000 0010 0001 (0x0021). `motor_flags` defaults to 0xFFFF (all motors) if not specified
`mode` can be 0 to enable immediately, or 1 to enable at the end of the current movement queue

#### <a name="cmdDisableMotors"></a>`disable_motors`
*`[uint16_t motor_flags, [int8_t mode (0x00, 0x01)]]`*

disable_motors can be called with optional paramters. It will make the specified motors become idle, i.e. unpowered and able to be freely moved.

`motor_flags` is a uint16 specifying which motors should be disabled. Byte 0 (the LSB) corresponds to motor 0, so to disable motors 0 and 5, you would send b0000 0000 0010 0001 (0x0021). `motor_flags` defaults to 0xFFFF (all motors) if not specified
`mode` can be 0 to disable immediately, or 1 to disable at the end of the current movement queue


#### <a name="cmdFallProtection"></a>`fall_protection`
*`bool enabled`*

Fall protection will disable all motors if it detects Marty is falling. It does this by measuring the Z axis of the accelerometer, and reacting when it passes a threshold. The accelerometer signal is digitally low pass filtered to try and prevent false positives. When fall protection is enabled, a change in fall state will be published on the chatter topic, and the servos_enabled will show deactivation of the servos.

`enabled` should be 1 to activate fall protection, or 0 to deactive it. Fall protection is disabled on startup.

It is stongly recommended that fall protection is activated when Marty is in use, as this can save your motors, especially if your robot falls from a height.


#### <a name="cmdMotorProtection"></a>`motor_protection`
*`bool enabled`*

Motor protection uses the build in current sensors on motors 0-7 to deactivate a specific motor if it senses it has become overloaded. This is on by default, and deactivating it should not be done unless you're really sure you want to. Deactivating motor protection will void the warranty on your motors.

`enabled` should be 1 to activate, or 0 to deactivate. Enabled on startup.


#### <a name="cmdLowBattery"></a>`low_battery_cutoff`
*`bool enabled`*

This will start beeping if the voltage on the battery starts to get too low (below 7.0v or 7.4v depending on settings). After a minute of prolonged undervoltage, it will disable the motors. This is enabled on startup.

Marty's control board has built in undervoltage cutoffs to protect LiPo batteries, and the supplied battery also has its own low voltage cutoff. However, due to large current draws when motors are driven, the battery voltage can fluctuate, and it is not advisable to let the robot reach these hard cutoffs. Therefore, the software low_battery_cutoff is recommended to be used.

`enabled` should be 1 to activate, or 0 to deactivate. Enabled on startup.


#### <a name="cmdBuzzPrevention"></a>`buzz_prevention`
*`bool enabled`*

This feature will attempt to stop servo 'buzz', which occurs when a servo is slightly away from its commanded position, but the motor signal isn't quite strong enough to move the output. It will adjust the commanded output position to try and match the real position. This feature is strongly recommended, as it'll help prolong the life of your servos, and make the robot sound better as it'll get itself comforable after completing a movement.

This feature also allows for manual movement of the joints, so you can push a joint to adjust its position.

_note that buzz prevention is not on the eyes, so don't try to move them manually_

`enabled` should be 1 to activate, or 0 to deactivate. Disabled on startup.


#### <a name="cmdSetIOType"></a>`set_IO_type`
*`uint8 io_number, int8 type`*

Used to set the type of GPIO port.

`io_number` is the port number, from 0-7
`type` can be 0 for digital input (default), or 2 for digital output.

More types will be supported in the future, but v1.0.0 of Rick can only support digital in/out.


#### <a name="cmdIOWrite"></a>`IO_write`
*`uint8_t io_number, float32 value`*

Change the state of an output pin. Pin must have already been configured with [`set_IO_type`](#cmdSetIOType).

`io_number` is the port number, from 0-7
`value` is a 4-byte float with the value to output.

It's a bit of overkill to use a float when currently only digital out is supported, but we'll be adding PWM out and potentially Analog out in the future, so we have float for futureproofing.


#### <a name="cmdI2CWrite"></a>`i2c_write`
*`uint8[n] address + data`*

Sends the data bytes over i2c. The first data byte should be the address, usual i2c parlance etc.
This will transmit over i2c1, the one connected to the accelerometer and broken out next to the GPIO pins.
Address 0x3A is the on board accelerometer, so avoid adding another slave device with this address.


#### <a name="cmdCircleDance"></a>`circle_dance`
*[`uint8 side`](#cmdDirectionCodes)`, `[`uint16 move_time`](#cmdMoveTime)*

Moves the hips and knees to move head in a circle. `side` specifies whether to start with left or right, and whether to go clockwise or anticlockwise. Will leave Marty leaning forwards, and you can send multiple commands to queue multiple circles.


#### <a name="cmdLifelike"></a>`lifelike_behaviours`
*`bool enabled`*

Lifelike behaviours will make Marty do something after a minute of inactivity, and every minute after that. Behaviours include tapping feet, swinging arms, looked angry, etc.
After each behaviour Marty's eyebrows will move to indicate battery voltage, he'll get angrier as his battery runs down.

`enabled` should be 1 to activate, or 0 to deactivate


#### <a name="cmdEnableSafeties"></a>`enable_safeties`
*-no parameters-*

Enable safeties will activate fall protection, buzz prevention, and increase the battery cutoff voltage. It should be called normally at the start of operation, as it puts Marty into the recommended operating state.


#### <a name="cmdSetParameter"></a>`set_parameter`
*`uint8 paramID, params`*

| Param ID | Data                |  Description  | 
|:---------|:--------------------|:--------------|
| 0        | `uint8 lean_amount` | Changes amount of side to side movement when walking etc. <br />leanAmount is a percentage of nominal, from 0-200. So setting leanAmount to 100 will revert to normal. |
| 2        | `uint16_t topicID, uint16_t period` | Rostopic publishing period <br />Period is specified in milliseconds<br />topicID is the rosserial topic ID, so: Accel: 104; GPIO: 106; Battery: 107; Motor currents: 108; Servo positions: 113 |
{:.tt}

#### <a name="cmdGetFirmwareVersion"></a>`get_firmware_version`
*-no parameters-*

Will cause the firmware version to be published on the chatter topic


#### <a name="cmdMuteESPSerial"></a>`mute_ESP_serial`
*-no parameters-*

Will make the ST micro’s serial lines for ESP comms go to high impedance. This is useful for two reasons:
 - To program the ESP
 - To stop the ESP being annoying when connected to a pi over the other serial port 

Note that calling this will make the control board unable to receive commands over wifi until the next reset.


#### <a name="cmdClearCalibration"></a>`clear_calibration`
*-no parameters-*

Clear servo calibrations, and calibrated flag from flash memory. Currently requires a power cycle before calibrations in RAM will be cleared


#### <a name="cmdSaveCalibration"></a>`save_calibration`
*-no parameters-*

Save current positions as zero position.
Note that _all_ servos must be enabled and in commanded positions for save calibration to work. Check the servo_positions topic to make sure none of the motors are at -100

Success or failure of calibration will be published over the chatter topic



`ROS COMMAND` Type Packets
---

The `ROS COMMAND` packet format is the lowest level of access offered by the TCP Socket API,
and simply passes on the `Data` byte array to the main controller chip via **ROS Serial**.
The size bytes are similar to the `COMMAND` packet format's size bytes, giving the length
of the `Data` array in bytes.

| Byte 0 | Byte 1     | Byte 2     | Byte 3   | ... | Byte M    |
|--------|------------|------------|----------|-----|-----------|
| 0x03   | *Size LSB* | *Size MSB* | Data[0]  | ... | Data[N-1] |
{:.bitfield}

More information on ROS (and ROS Serial) is given [here](/ros/)

<br>


Simple Sockets Example in Python 3
===


Just to illustrate how the socket API works, though if you're using Python to communicate
with Marty  we'd recommend you check out the **MartyPy** api [documented here](/python/remote/)
through which you can access lower-level socket stuff if you want (in the `Marty.client.sock`
member)

{% highlight python %}
import socket
import struct

ip = '192.168.0.42' # Adjust accordingly
port = 24

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((ip, port))

# Send the Accelerometer x Axis Command
sock.send(''.join([chr(b) for b in [0x01, 0x02, 0x00]]).encode('ascii'))

accel_raw = sock.recv(4)
accel = struct.unpack('f', accel_raw)[0]
{% endhighlight %}


<br>
<br>

------------------------------------------------------------------------------------------------


Other Endpoints
===


Over Ports 80 & 4000
---
{:.subhead}


`http://______:80/service-discovery`
{:.docitem}

A Marty will respond with a `HTTP 200 OK` with `AA` in the body, followed by the
Robot's name (if configured). The `______` part should be replaced with the Marty's
IP address as appropriate.

This can be used for 'brute force' discovery of robots by sending the same request to
all addresses in a range you expect a Marty to be on.

Note that this may not play well with firewalls or routing policies.



`http://______:80/`
{:.docitem}

The board will drop into a special 'hotspot mode' if it can't connect to a
Wireless network. If you then connect to this network (called 'Marty Setup' followed
by some digits) you'll be presented with the config page.

This enpoint will only be exposed when wither 1) the Marty cannot connect to a WiFi
network or 2) you press **Bob the Button**, which will make a noise and bring the hotspot
up.


`UDP port 4000`
{:.docitem}

Martys will respond to a Multicast UDP packet `AA`, giving their name and IP.
Sockets can be fiddly and multicast can be particularly fiddly, so mileage will
vary based on a combination of operating system, local network settings and hardware.
The multicast address can also vary.

As a minimal example, this works for us in Python 3:

{% highlight python %}
import socket

socket_addr = "224.0.0.1"
socket_port = 4000
magic_command = b"AA"

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 32)
sock.sendto(magic_command, (socket_addr, socket_port))

while True:
    data, addr = sock.recvfrom(1000)
    print("{}: {}".format(addr, data))
{% endhighlight %}

<br>
<br>
