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
over port `24`.
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
---

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
in length depending on the operation, so the 1st and 2nd bytes encode the payload size
(number of *bytes*), though don't count themselves nor the zeroth byte in the size.

| Byte 0 | Byte 1     | Byte 2     | Byte 3           | Byte 4    | ... | Byte M    |
|--------|------------|------------|------------------|-----------|-----|-----------|
| 0x02   | *Size LSB* | *Size MSB* | *Command Opcode* | *1st Arg* | ... | *Nth Arg* |
{:.bitfield}

The payload size (1st and 2nd bytes) should be encoded as a little endian integer.
Bytes in position 4 through M depend on the opcode and command being called.



| Command                | Opcode     | Size | Arguments                                               |
|:-----------------------|:-----------|:-----|:--------------------------------------------------------|
| *hello*                | 0x00       | 1    | *-none-*                                                |
| *lean*                 | 0x01       | 5    | uint8 dir, uint8 amount, uint16 move\_time              |
| *walk*                 | 0x03       | 7    | uint8 steps, uint8 turn, uint16 move\_time, int8 step\_length, int8 side |
| *kick*                 | 0x05       | 5    | uint8 amount, int8 twist, uint16 move\_time             |
| *celebrate*            | 0x08       | 3    | uint16 move\_time                                       |
| *arms*                 | 0x0B       | 5    | int8 r\_angle, int8 l\_angle, uint16 move\_time         |
| *sidestep*             | 0x0E       | 6    | int8 side, int8 num\_steps, uint16 move\_time, int8 step\_length  |
| *stand straight*       | 0x0F       | 3    | uint16 move\_time                                       |
| *play\_sound*          | 0x10       | 7    | uint16 freq\_start, uint16 freq\_end, uint16 duration   |
| *stop*                 | 0x11       | 2    | uint8 stop\_type (0x00, 0x01, 0x02, 0x03)               |
| *move\_joint*          | 0x12       | 5    | uint8 joint\_id, int8 position, uint16 move\_time       |
| *enable\_motors*       | 0x13       | ??   | **TODO: Now takes args**                                |
| *disable\_motors*      | 0x14       | ??   | **TODO: Now takes args**                                |
| *fall\_protection*     | 0x15       | 2    | bool enabled (0x00 or 0x01)                             |
| *motor\_protection*    | 0x16       | 2    | bool enabled (0x00 or 0x01)                             |
| *low\_battery\_cutoff* | 0x17       | 2    | bool enabled (0x00 or 0x01)                             |
| *buzz\_prevention*     | 0x18       | 2    | bool enabled (0x00 or 0x01)                             |
| *save\_calibration*    | 0xFF       | 1    | *-none-*                                                |
{:.tt}

<br>




`ROS COMMAND` Type Packets
---

The `ROS COMMAND` packet format is the lowest level of access offered by the TCP Socket API,
and simply passes on the `Data` byte array to the main controller chip vie **ROS Serial**.
The size bytes are similar to the `COMMAND` packet format's size bytes, giving the length
of the `Data` array in bytes.

| Byte 0 | Byte 1     | Byte 2     | Byte 3   | ... | Byte M    |
|--------|------------|------------|----------|-----|-----------|
| 0x03   | *Size LSB* | *Size MSB* | Data[0]  | ... | Data[N-1] |
{:.bitfield}

More information on ROS (and ROS Serial) is given [here](/ros/)

<br>

### Simple Sockets Example in Python 3

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
