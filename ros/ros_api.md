---
title: ROS API
breadcrumbs:
  - name: 'ROS'
    url: '/ros'
layout: article
---

Initialisation
===

On boot, Marty will call a startup script which initialises the required ROS parameters, such as the required subscriber/publisher handles.
From here, it will then initialise the core command server, calibrate the servos and open a port for handling incoming connections (for example from a remote computer).

<div>

<br class="smaller">
<a href="/ros/ros_marty_doc/html/index.html" class="inherit">
<div class="well bigger">
ros_marty Documentation
<div class="smaller muted">
Here you can find the documentation for the ros_marty stack
</div>
</div>
</a>

<br class="smaller">
<a href="/ros/marty_msgs_doc/html/index-msg.html" class="inherit">
<div class="well bigger">
  marty_msgs Documentation
<div class="smaller muted">
  Documentation for the messages used by Marty's ROS stack
</div>
</div>
</a>

<br class="smaller">
</div>

ROS topic map
===

<img src="/ros/rosgraph.svg" alt="ROS graph" style="width: 700px;"/>
<!-- ![ROS nodes](/ros/rosgraph.svg =100x200) -->
