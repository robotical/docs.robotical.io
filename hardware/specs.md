---
title: Hardware Specifications
breadcrumbs:
 - name: Home
   url: '/'
 - name: 'Hardware'
   url: '/hardware'
layout: article
---

Rick - Ratings and Maximums
---

Rick has a maximum power output of **3A** per each of the two **5v** channel, maximum operational
temperature of **50&deg;C**, thermal cut-out at **90&deg;C** and an on-board dual-cell
**7.4v LiPo** battery charger with **\_\_A** peak charge current. Nominal battery voltage is
between **6.4v** and **18v**. A discharged battery below **4.15v** will not be charged by the board.
Nominal charge voltage is between **8v** and **12v**. The two 5v regulators will be disabled
when the battery voltage is below **6.4v**, powering Rick off. 
{:.feature}

Rick has three on-board power regulators: two `ST1S10` switch-mode 5v regulators,
and one `LD3905PU33R` 3v3 LDO regulator. The battery charge IC is a `mcp73213` dual-cell
BCM Controller. See each manufacturer's datasheet for exact ratings and capabilities.

One of the `ST1S10` regulators powers the servo motor channels directly on the `M0` through
`M8` pin-headers on Rick, with the name <code>+5V<sub>M</sub></code>.
Its 3 Amp capacity applies solely to the load on these motor channels, independently of any
loads attached to the `GPIO` pins. The nominal voltage in our testing is **5.2v**,
(&plusmn;0.2v). The voltage between the <code>+5V<sub>M</sub></code> pins and
<code>M<sub>GND</sub></code> pins may be slightly less than this as each motor channel
*except* `M8` has a **0.1&#x2126;** current sense resistor between it and `GND`.

Do not connect the Motor's <code>M<sub>GND</sub></code> pins to the `GND` pins elsewhere
on Rick or any other attached, grounded device as this may damage the control board. This
is because of the current sense resistors.
{:.alert.danger.tag.padder}

Rick has some *basic* reverse-polarisation protection, with two Q-FETs
that prevent current flow when either the Battery `BAT` or Charger `CHG` are
connected the wrong way around through the **4 pin JST** connector `P5`. **However**
the charging chip `U7` is **NOT** protected and **will be destroyed if the board is
reversely polarised.** If you're not using the supplied connectors then take care to
check polarisation.
{:.alert.info.tag.padder}

The reverse polarity protection is also not designed to protect against sustained or
out-of-nominal voltages in a reverse-polarised mode, i.e. anything above **8v**.
The protection is provided by two `NTR4101PT1G` Q-FETs, `Q1` and `Q2`.

The 5v regulators have internal thermal-cutouts to protect against sustained high loads.
They will cut out when at **90&deg;C**, dissipating a *lot* of power. Be careful and do
not touch the PCB if driving high loads as it can cause injury. The Battery Charge chip
also has a built in thermal cutout at **85&deg;C**. Frequent or sustained high loads
could also permanently damage your control board.
{:.alert.danger.tag.padder}



Charge Cable and Supplied Battery
---

The supplied USB to 9v barrel-jack is rated for **5v** on the USB end, and outputs
**9v** on the barrel jack, centre pin positive. The supplied rechargeable LiPo battery
has a nominal charged voltage **8.4v**. The battery's capacity is **1400mAh**.
{:.feature}

The supplied cable can provide adequate voltage to charge the battery through rick, but
using an equivalent mains-powered 9v DC charger the battery can charge faster *at your own risk*.
For the fastest charging an external LiPo battery charger, commonly used for Remote-Control applications
can be used to charge the battery *again, at your own risk*.

Lithium polymer batteries (LiPo) can be dangerous if abused. Do not overcharge the battery
or discharge it below the minimum safe voltage of **6.4v**. Don't disassemble or pierce the
battery, leave it charging unattended or dispose of it in regular household waste or by burning.
{:.alert.danger.tag.padder}
