---
title: Sensors
layout: article
breadcrumbs:
 - name: 'Hardware'
   url: '/hardware'
---

On-Board Sensors
----------------

[Rick](/hardware/rick) has some integrated sensors that can be accessed through it's [API](/hardware/sicket-api)

Acclerometer
: [MMA8452Q 3-axis, 12/8 bit accelerometer](https://public.robotical.io/Datasheets/MMA8452QT.pdf) connected on the 1st I<sup>2</sup>C bus.
By convention, the axes of the accelerometer are re-mapped in the API from their manufacturer assignments onto the standard robot frame of reference
(x forward, y laterally left, z vertically up). These are marked on the PCB.

Motor Current (*"Force"*) Sensors
: A 0.1 Ω 1206 resistor (Schematic ref. R7 to R14) is attached from the Servo low-side to ground, and then to a GPIO on U1
for all motor channels *except* channel 9 (used for the Eyes). This permits motor current sensing, as a proxy for exertion/force/torque.


Add-On Sensors
==============

**Note that** add-ons such as the [Raspberry Pi Camera](https://robotical.io/shop/sku/20) don't interact with Rick
so aren't documented here.


<a name="vcnl4200"></a>

### VCNL4200 Distance Sensor Add-on

![VCNL4200 Distance Sensor Board](https://content.robotical.io/static/media/stock/DSC07892-ed-opt.jpg)


VCNL4200 Module Datasheet
: [https://public.robotical.io/Datasheets/VCNL4200.pdf](https://public.robotical.io/Datasheets/VCNL4200.pdf)

I<sup>2</sup>C Address
: `0x51`

Voltage
: 5V input, logic 3V3 (*not* 5V tolerant)

Product Schematic
: [https://public.robotical.io/Schematics/VCNL4200_Board/VCNL4200_breakout_schematic.pdf](https://public.robotical.io/Schematics/VCNL4200_Board/VCNL4200_breakout_schematic.pdf)

Dimensions
: 26mm &times; 13.5mm &times; 13mm (with connector)

Cable & Connector
: Approx 22cm long, JST XH 5 pin (10mil spaced) on board to 2 &times; 3 female pin header


<br>
