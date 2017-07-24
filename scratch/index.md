---
title: Scratch
layout: article
---

Scratch is the simplest and easiest way to start programming your Marty,
if you're new to programming. The interface lets you graphically build up
a program from blocks.
{:.feature}


<div class="center bitbigger padder">
<a href="http://scratchx.org/?url=http://robotical.github.io/scratchx/scratch_extensions/martyExtended.js#scratch" class="btn rounded" target="_blank"><i class="fa fa-external-link"></i> &nbsp; Click to open Scratch &times; Marty</a>
</div>

Scratch needs to connect to Marty over your WiFi network to send instructions, so Marty must be on your WiFi network. If you haven't already, then check out the [WiFi setup guide](/learn/article/Marty%20Setup%2C%20Calibration%20%26%20Troubleshooting%20Guide/WiFi%20Setup)

When you open Scratch, it'll try and find Martys on your network. Have a look at the [Scratch getting started guide](/learn/article/Get%20Started%20with%20Scratch/Introduction) for details on how that works.

<div class="center bitbigger padder">
<a href="/learn/article/Get%20Started%20with%20Scratch/Introduction" class="btn rounded" target="_blank"><i class="fa fa-external-link"></i> &nbsp; Getting started with Scratch</a>
&nbsp;&nbsp;
<!--
TODO: LINK TO TUTORIALS / LEARNING MATERIALS ETC.
-->
</div>

## Useful info

It's also worth checking out the [Intro to Marty Behaviour](/learn/article/Intro%20to%20Marty%20Behaviour/Introduction) article for details on how Marty behaves and reacts. But in brief

 * Marty's motors are turned off by default, they're enabled using the `Get Ready` block (which will also move Marty to the zero position), or the `Enable motors` block.
 * Fall protection will turn off Marty's motors if it looks like Marty is falling over
 * Motor over-current protection will deactivate a motor if it gets a large instantaneous load, or a sustained heavy load
 * Buzz prevention will try to reduce the torque on Marty's motors when movements are finished - so Marty might move a little bit to get comfy. It'll also let you move his motors (except for the eyes) by hand, if you push gently.

## Function Reference

This section will give details on each block available through the Scratch interface.

| Action Blocks                     | Sensor Blocks           | Marty Selector         |
|:----------------------------------|:------------------------|:-----------------------|
| [Circle dance](#circle_dance)     | Accelerometer           | Select Marty           |
| [Enable motors](#enable_motors)   | Battery Voltage         | Select Marty on IP     |
| [Eyes](#eyes)                     | Input                   |                        |
| [Get Ready](#get_ready)           | Motor current           |                        |
| [Kick](#kick)                     | TODO: motor position    |                        |
| [Lean](#lean)                     | TODO: motor enabled     |                        |
| [Lift leg](#lift_leg)             |                         |                        |
| [Lower leg](#lower_leg)           |                         |                        |
| Move joint                        |                         |                        |
| Move leg forward/backward         |                         |                        |
| Play sound                        |                         |                        |
| Set blocking mode                 |                         |                        |
| Stand straight                    |                         |                        |
| Stop                              |                         |                        |
| Turn                              |                         |                        |
| Turn off motors                   |                         |                        |
| Walk                              |                         |                        |
| Walk backward                     |                         |                        |
| Walk forward                      |                         |                        |
| Wiggle                            |                         |                        |
{:.tt}

<a name="circle_dance"></a>
#### Circle Dance ![Circle dance block](/img/blocks/circle_dance.png)

`Circle dance` will make Marty lean right->forward->left->backward, or left->backward->right->forward, depending on the setting. His head will therefore move round in a clockwise or anticlockwise direction.

The time is for a complete cycle.

The movement will end with Marty leaning either forward or backward, so you can send several `circle dance` commands in a row to produce a continuous motion

<a name="enable_motors"></a>
#### Enable Motors ![Enable motors block](/img/blocks/enable_motors.png)

`Enable motors` will activate Marty's motors so they're ready to receive commands, but won't send any movement commands.

`Enable motors` will also un-pause any queued movements, so if you've previously used the [`stop and pause`](#stop) block, you can use `enable motors` to resume movement.

<a name="eyes"></a>
#### Eyes ![Eyes block](/img/blocks/eyes.png)


The `Eyes` block will let you move Marty's eyebrows near-instantly to a predefined position - either *angry*, *normal*, *excited*, or *wide*

<a name="get_ready"></a>
#### Get Ready ![Get ready block](/img/blocks/get_ready.png)


The `Get ready` block is designed to get Marty's motors activated and finish with all joints in their zero positions. It'll also wiggle the eyebrows, so you know that it's done something even if Marty was already standing straight.

It will [`enable motors`](#enable_motors)

It's a good idea to call this at the beginning of a script, to make sure Marty's joints are enabled and in a known position. 

When Marty's motors have been deactivated, he has no way to tell where they are. So **when you call `Get ready` Marty may move very quickly** and jerk a bit. Keep fingers clear!


<a name="kick"></a>
#### Kick ![Kick](/img/blocks/kick.png)


`Kick` will make a kicking movement with Marty's right or left legs.

TODO: Add time and twist parameters for extended interface

<a name="lean"></a>
#### Lean ![Lean](/img/blocks/lean.png)


Lean will make Marty lean forward, backward, left or right, in the given number of seconds. 

Forwards and backwards will move Marty's hip joints together, while left and right will move Marty's knee joints.

<a name="lift_leg"></a>
#### Lift leg ![Lift leg block](/img/blocks/lift_leg.png)

`Lift leg` will move either Marty's left or right knee outward to lift that leg up. This might make Marty tilt sideways, depending on his current position

<a name="lower_leg"></a>
#### Lower leg ![Lower leg block](/img/blocks/lower_leg.png)

`Lower leg` will measure which of the two knees is higher up, and then lower it to the same angle as the lower knee. This will have the effect of lowering a raised leg to the ground.

This function currently doesn't take into account hip angles, so you might get unexpected behaviour if the hip angles don't have the same magnitude.


<a name="move_joint"></a>
#### Move joint ![Move joint block](/img/blocks/move_joint.png)

<!-- table of blocks with links -->
<!-- how to read a sensor -->






