---
title: Tutorial
breadcrumbs:
  - name: 'V-REP Simulation'
    url: '/vrep'
layout: article
---

Initialising
===

Assuming your <a href="/vrep/getting_started">installation</a> went well, Marty
should now be able to ROS around in simulation!

***Note:*** in order for the interface to be loaded correctly, a `roscore` ***must***
be running somewhere, preferably one on your machine set to a localhost.

Before we get started, let's first check everything installed as planned. From a
terminal, navigate to your V-REP root folder and run `vrep.sh`. This will launch
V-REP but also show the status of the plugins that are being loaded within the
terminal. If the interface installation was successful, V-REP will detect the plugin
and will output `Plugin 'RosInterface': load succeeded` to the terminal.

Now, in the top left corner click `File/Open Scene...` and from the list open
`rosInterfaceTopicPublisherAndSubscriber.ttt`. This is the ROS demo as provided by
V-REP and is designed to show the operation of a simple Publisher/Subscriber setup.
Feel free to examine the source code if you wish. Next, click the Play button located
on the toolbar. Your vision sensor should start sweeping the scene and display two
video feeds, one that is being published and one that is being received, as shown below.

![Alt text](/vrep/vrep_ros_test.png?raw=true "ROS test scene")

Marty the Simulated Robot
===

***Note:*** the `ros_marty` stack must be executed and set to simulation mode before
running V-REP for Marty to ROS around.

Let's open a new scene (`File/New Scene`) and see what Marty can do.

Within the `Model browser`, under `robots`, click `mobile`. This will list V-REP's
selection of mobile robots. Now, on the bottom left, scroll down and drag Marty
on to the scene.

![Alt text](/vrep/marty_drop.png?raw=true "Drag and drop")

Now, click Play. Go ahead and open up a new terminal and enter `rostopic list`.
A list of Marty's visible topics should be shown.

As a final test, we can try publishing commands to our simulated Marty:

{:.feature}
    rostopic pub /marty/servo marty_msgs/ServoMsg "servo_id: 6
    servo_cmd: 60"
{:.feature}

This should move Marty's left arm.

Congrats, Marty is now officially ROSing around! 

***Tip:*** use Tab autocompletion to save time and errors typing long ROS commands
in the terminal.
