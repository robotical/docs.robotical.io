---
title: Specifications
layout: article
breadcrumbs:
 - name: Home
   url: '/'
 - name: 'Hardware'
   url: '/hardware'
#versions:
# - name: 'rick'
#   value: '1.0.0'
---

Ratings and Maximums
---

Rick has a maximum power output of **3A** per **5v** channel, maximum operational temperature of
**50&deg;C**, thermal cutout at **85&deg;C** and an onboard dual-cell **7.4v LiPo** battery charger
with **\_\_A** peak charge current. Nominal battery voltage is between **\_\_v** and **\_\_v**.
{:.feature}

Rick has three onboard power regulators: two `ST1S10` switch-mode 5v regulators,
and one `LD3905PU33R` 3v3 LDO regulator. The battery charge IC is a `mcp73213` dual-cell
BCM Controller. See each manufacturer's datasheet for exact ratings and capabilities.

One of the `ST1S10` regulators powers the serov motor channels directly on the `M0` through
`M8` male pin-headers on Rick, with the name <code>+5V<sub>M</sub></code>.
Its 3 Amp capacity applies solely to the load on these motor channels, independently of any
loads attached to the `GPIO` pins. The nominal voltage in our testing is **5.2v**,
(&plusmn;0.2v). The voltage between the <code>+5V<sub>M</sub></code> pins and
<code>M<sub>GND</sub></code> pins may be slightly less than this as each motor channel apart
from `M8` has a **0.1&ohm;**
