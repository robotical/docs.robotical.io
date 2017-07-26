---
title: ROS Topic List
breadcrumbs:
  - name: 'V-REP Simulation'
    url: '/vrep'
layout: article
---


## Topic List

    /marty/servos         # Type: marty_msgs/ServoMsg
    /marty/servo_array    # Type: marty_msgs/ServoMsgArray
    /marty/accel          # Type: marty_msgs/Accelerometer

<div class="alert warning padder tag">
<p> There is a known issue with the simulated accelerometer, so the data coming from <code>/marty/accel</code> is inconsistent with the ROS stack.
Before the changes can be made official and changed by Coppelia, you can change <code>line 37</code> in the <code>Accelerometer</code> child script within V-REP to: -

    <br><br><code>accel={force[3]/(9.81*mass),force[1]/(9.81*mass),force[2]/(9.81*mass)}</code>

<br>    
</p>
</div>
