---
title: Getting Started
breadcrumbs:
  - name: 'ROS'
    url: '/ros'
layout: article
---

Installation
===

To install the latest version of ROS on your computer, follow the official ROS [guidelines](http://wiki.ros.org/ROS/Installation).
{:.feature}

Assuming a full version ROS has been installed, the following instructions should
get the ROS stack up and running.
{:.feature}

**Note:** currently only Linux is supported.

The following instructions are to be performed both on the ***Raspberry Pi*** within Marty and preferably on ***another computer***. In theory, you can develop code on the Marty itself, however in practice it is easier to develop on a separate machine and run using a [remote roscore](http://wiki.ros.org/ROS/Tutorials/MultipleMachines).

First of all, open a terminal. Now, let's initialise a catkin workspace. This can be anywhere but for simplicity's sake, let's place it in your `home` directory:


    mkdir ~/catkin_ws

Next, we need to create a folder to store our source code, this is where we'll
put our packages:

    mkdir ~/catkin_ws/src

Now, navigate to it by entering `cd ~/catkin_ws/src`.
Once there, initialise a catkin workspace:

    catkin_init_workspace

Alright, now we've set up the workspace! Next on the list is downloading and compiling the packages we need.

You should already be in the `src` folder. Here, we'll clone the repositories we need from GitHub:


    git clone https://github.com/robotical/ros_marty.git
    git clone https://github.com/robotical/marty_msgs.git
    git clone https://github.com/robotical/marty_description.git
    # These two are optional demos and extra functionality
    git clone https://github.com/robotical/marty_joy.git
    git clone https://github.com/robotical/marty_football.git


Now, `cd ..` back up into `catkin_ws`, and here, build the downloaded packages:

    catkin_make


If all goes well, the packages will have been built from source.

Next, we need to source the `setup` file:

    source devel/setup.bash


# Running

Before ROS can be used, your Marty has to be initially calibrated. Bundled within this package is a ready made calibration script, allowing individual servo control. To run this, execute the calibration script on the Marty:

    roslaunch ros_marty calibration.launch


Follow the on screen instructions and make sure to save the file when you're finished calibrating. This file will be used when Marty boots up or when zeroing.

Now, Marty's core services can be started:

    roslaunch ros_marty startup.launch


If all goes well, ROS should be up and running on the Marty.

Check the list of topics:

    rostopic list

If Marty is advertising the topics listed in the docs, then he's good to go.

Try publishing a servo message:

    rostopic pub /marty/servo marty_msgs/ServoMsg "servo_id: 7
    servo_cmd: 60"


Setting up remote communication
===

The easiest way to work with Marty is to set up your machine to access a <a href="http://wiki.ros.org/ROS/Tutorials/MultipleMachines">remote roscore</a>. The idea here is to run the ROS master on the Marty, and allows the PC to visualise data or send commands to the Marty over the network.
{:.feature}


Add your Marty's local IP address and hostname to your computer's *hosts* file located in `/etc`. The easiest way to do this would be through the terminal:

    $ sudo nano /etc/hosts

Your *hosts* file should look something similar to this:

    127.0.0.1	localhost
    127.0.1.1	MyComputer
    # INSERT MARTY CREDENTIALS HERE

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters

Insert your marty's local IP and hostname in the same format as the existing hosts. For example, given a Marty's IP address of `192.168.0.12` and a name of `marty2017`:

    127.0.0.1	localhost
    127.0.1.1	MyComputer
    192.168.0.12    marty2017

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters


**Note:** The reverse must be done on the Marty to ensure he can publish data to the remote PC.


Quality of life improvements
===

On your computer, you may wish to add the following to your `.bashrc` file:

    export ROS_MASTER_URI=http://192.168.0.12:11311
    source /opt/ros/kinetic/setup.bash
    source ~/catkin_ws/devel/setup.bash

Every time a new terminal is opened, this sets the `ROS_MASTER_URI` of your PC to point to the Marty, and in addition sources the correct setup files. Of course, if your Marty has a different IP address or your workspace is called something other than `catkin_ws`, these will have to be changed accordingly.

This can be ommitted, though you would need to type in the above to every new terminal window you want to receive ROS information from the Marty.
