---
title: Rick
layout: article
breadcrumbs:
 - name: 'Hardware'
   url: '/hardware'
versions:
 - name: 'Rick Control Poard PCB'
   value: 'v.1.0.0'
---

**TODO:** Board layout, serial, I2C and WiFi interfaces
{:.text-danger}

Schematics
===

<div>
<br>
<div class="well">
<p class="bitbigger">
<span class="spacer"></span>
<i class="fa fa-file-pdf-o fa-fw"></i>
<span class="spacer"></span>
<a href="{{ site.baseurl }}/img/rick-schematic-v1.0.0.pdf" class="inherit">
Full Colour PDF schematic of <strong>Rick v1.0.0</strong>
</a>
</p>
</div>
<br>
<div class="well">
<p class="bitbigger">
<span class="spacer"></span>
<i class="fa fa-file-pdf-o fa-fw"></i>
<span class="spacer"></span>
<a href="{{ site.baseurl }}/img/rick-schematic-v1.0.0-mono.pdf" class="inherit">
Monochrome PDF schematic of <strong>Rick v1.0.0</strong>
</a>
</p>
</div>
</div>


<h2>Translations</h2>

<p>
  Some nets are marked slightly differently between the silkscreen on Rick and in the schematic.
</p>

<table class="table bitbigger">
  <thead>
    <th>Shown on Schematic</th>
    <th>Shown on the PCB</th>
  </thead>
  <tr>
    <td><code>VMTR</code> is one of the two 5 volt busses, specifically for the motors</td>
    <td><code>+5V<sub>M</sub></code></td>
  </tr>
  <tr>
    <td><code>SERVO_<i>n</i>S</code> Are the <i>n</i> servo current sense nets</td>
    <td><code>M<sub>GND</sub></code></td>
  </tr>
  <tr>
    <td><code>SERVO_<i>n</i>C</code> Are the <i>n</i> servo control/signal nets</td>
    <td><code>SIG<i>n</i></code></td>
  </tr>
  <tr>
    <td><code>RX_1, TX_1</code> Are the main serial pins</td>
    <td><code>RX, TX</code></td>
  </tr>
  <tr>
    <td><code>SDA_1, SCL_1</code> Are the main I<sup>2</sup>C pins</td>
    <td><code>SDA<sub>1</sub>, SCL<sub>1</sub></code></td>
  </tr>
  <tr>
    <td><code>SDA_2, SCL_2</code> Are the secondary I<sup>2</sup>C pins</td>
    <td><code>SDA<sub>2</sub>, SCL<sub>2</sub></code></td>
  </tr>
  <tr>
    <td><code>CHG_IN</code></td>
    <td><code>CHG+</code></td>
  </tr>
  <tr>
    <td><code>BATT_IN</code></td>
    <td><code>BATT+</code></td>
  </tr>
  <tr>
    <td><code>GND</code> Common ground</td>
    <td><code>CHG-, BATT-</code></td>
  </tr>
</table>


<br>
