---
title: Getting Started
breadcrumbs:
  - name: 'V-REP Simulation'
    url: '/vrep'
layout: article
---

{:.feature}
V-REP allows the use of ROS through a dedicated interface, `RosInterface`, which
is part of the <a href="http://www.coppeliarobotics.com/helpFiles/en/apisOverview.htm">V-REP API framework </a>.
By duplicating the C/C++ ROS API, V-REP can act as a ROS node and enables
communication via ROS topics.

In order to do this, the interface has to be installed.
{:.feature}

Installation
===

***Note:*** To proceed, <a href="/ros/getting_started">ROS</a> and the
<a href="https://github.com/robotical/marty_msgs">`marty_msgs`</a> package *must*
be installed.

To install, please follow *Coppellia's*  `RosInterface` installation instructions
located in `/$VREP_ROOT/programming/ros_packages`.

Before compiling the interface, you have to tell V-REP that you will be using
custom messages. This is relatively easy to do and can be done by adding the
desired messages to the `messages.txt` file located in the `meta` folder of your
`RosInterface` folder, ***prior*** to compiling the interface.

In our case:

{:.feature}
    marty_msgs/ServoMsg
    marty_msgs/ServoMsgArray
    marty_msgs/Accelerometer
{:.feature}

Entering Simulation Mode
===

Now that the interface has been installed, the `ros_marty` package has to be
set to simulation mode. Two files have to be edited: `config.cfg` and `marty.launch`.

Let's do `config.cfg` first. This file is located within `../ros_marty/cfg` and,
thankfully, the change to be made is simple. Set the `false` variable to `true`:

{:.feature}
    simulated: false     # Whether Marty is being simulated
{:.feature}

Next, let's edit `marty.launch`. This file is located in the `launch` folder of
your `ros_marty` package. Again, the change is relatively simple, set the `false`
variable to `true`:

{:.feature}
    <param name="use_sim_time" value="false"/> <!-- Enable if Simulated -->
{:.feature}

Now, instead of using a "real-time" clock, it will use the
simulator's timing system. This is important as V-REP can be sped up or slowed down.
