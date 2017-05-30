---
title: Low Level APIs
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


TCP Socket API
===

The `ESP8266` WiFi modem on Rick presents a low-level TCP socket API to the network. Clients
connecting to this API can send byte-level commands to the ESP which will in turn instruct
the main control chip to perform an action or report back on a sensor. The socket API runs
over port `24`
{:.feature}


`GET` Type Packets
---

These `GET` requests fetch sensroy information. A `GET` request is characterised by the zeroth
packet byte being `0x01`. The 2nd byte then selects the sensor type, and the third selects
the ID, if applicable. Where the Type is undefined, any value may be sent (e.g. `0x00`) but
it must be there so the packet length is correct. All `GET` packets are of length 3.

| Byte 0 | Byte 1  | Byte 2  |
|--------|---------|---------|
| 0x01   | *Type*  | *ID*    |
{:.bitfield}

Currently there are 4 types of sensor queryable: Battery voltage, Accelerometer, Motor Currents and GPIOs.


| Sensor           | Type Byte  | ID Byte           | ID Description      |
|:-----------------|:-----------|:------------------|:--------------------|
| *Battery*        | 0x01       | *(undefined)*     | *Not applicable*    |
| *Accelerometer*  | 0x02       | [0x00, ..., 0x02] | x, y and z axes     |
| *Motor Current*  | 0x03       | [0x00, ..., 0x07] | Motors 0 through 7  |
| *GPIO*           | 0x04       | [0x00, ..., 0x07] | GPIOs 0 through 7   |
{:.tt}

All readings returned after a `GET` request are a 4 Byte Little-Endian float, i.e. the
LSB is first, the MSB last.

The 8th motor channel (usually the eyes on Marty) does *not* have a current sensor
so no value can be reported.

<br>


`COMMAND` Type Packets
---

Command packets intstruct the control board to do something. Similar to the `GET` type packets,
`COMMAND` packets all begin `0x02`. This type of packet can vary
in length depending on the operation, so the 1st and 2nd bytes encode the payload size, though
don't count themselves nor the zeroth byte in the size.

| Byte 0 | Byte 1     | Byte 2     | Byte 3           | ...       | Byte M    |
|--------|------------|------------|------------------|-----------|-----------|
| 0x02   | *Size LSB* | *Size MSB* | *Command Opcode* | *1st Arg* | *Nth Arg* |
{:.bitfield}

The payload size (1st and 2nd bytes) should be encoded as a little endian integer.
Bytes in position 4 through M depend on the opcode and command being called.



| Command          | Opcode     | Size | Arguments                                               |
|:-----------------|:-----------|:-----|:--------------------------------------------------------|
| *hello*          | 0x00       | 1    | *none*                                                  |
| *lean*           | 0x01       | 5    | uint8 dir, uint8 amount, uint16 move\_time              |
| *walk*           | 0x01       | 7    | uint8 steps, uint8 turn, uint16 move\_time, int8 step\_length, int8 side |
{:.tt}

**TODO:** Complete all `COMMAND` opcodes
{:.bigger.text-danger}

<br>


`ROS COMMAND` Type Packets
---

**TODO**
{:.bigger.text-danger}

<br>

### Example Socket Call in Python 3

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
