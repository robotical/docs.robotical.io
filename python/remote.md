---
title: MartyPy Library
breadcrumbs:
  - name: 'Python APIs'
    url: '/python'
layout: article
---

Installation
===

The easiest way to install *martypy* is using `pip`, the Python package manager:
{:.feature}

    $ pip install martypy
{:.feature}

Also, when starting a new Python project it is **strongly** recommended that you start it
using a *virtual environment*, with `pyvenv` for Python 3 like so:
{:.feature}

    $ pyvenv VENV
    $ source VENV/bin/activate
    (VENV) $ pip install martypy
{:.feature}

Instead, for Python 2 (although we recommend you don't start using Python 2 for
new projects) use `virtualenv`:

    $ virtualenv VENV
    $ source VENV/bin/activate
    (VENV) $ pip install martypy

This prevents you from having to install martypy for *all* users on your computer, and lets you
install specific versions of packages without having conflicts with system packages.

The source code form `martypy` is on [GitHub](https://github.com/robotical/martypy), if you need to
install the bleeding-edge in-development code, or want to fiddle with the library, submit issues
or contribute pull requests.


**ProTip:** See [Python.org](https://www.python.org) for how to install Python on your
System, and for generic syntax help and learning resources
{:.alert.success.tag}


Basic Usage
===

The `martypy` library has a core client class, `Marty` that provides
all the methods used to control a robot. The class takes one required argument,
which is a URL-like string that lets it know where to find your Marty.

{% highlight python %}
>>> from martypy import Marty
>>> mymarty = Marty('socket://marty.local')
>>> mymarty.hello()  # Move to zero positions and wink
True
>>> mymarty.playsound(8000, 200, 1000) # Play a tone
{% endhighlight %}

If you're using the Marty remotely, over WiFi, that'll be `socket://` followed by the **IP Address**
of your robot, or a resolvable name, like `socket://192.168.0.42` or `socket://marty.local`. If your
robot can't be found, this step may throw an Exception.

<br>





<a name="main-members"></a> Main Functions and Members
===



`martypy.Marty(url='socket://marty.local', client_types=dict(), *args, **kwargs)`
{:.docitem}

Class constructor for a Marty client instance, with a default URL given.

`*args` and `**kwargs` are passed on to the client type, which will be chosen depending on the protocol
specified the URL. Currently the supported client types are `socket` and `test`.

For more info on extending the available client types via `client_types` see [here](#client-types).



`hello()`
{:.docsubitem#hello}

Moves to the zero position and wiggles the eyebrows



`stop(stop_type=None)`
{:.docsubitem#stop}

Stop the robot moving. `stop_type` is a str which should be a key in the `Marty.STOP_TYPE` dict.
If it is none (the default) then `clear and zero` will be assumed. Other options are:

clear queue
: clear movement queue only (so finish the current movement)

clear and stop
: clear movement queue and servo queues (freeze in-place)

clear and disable
: clear everything and disable motors

clear and zero
: clear everything, and make robot return to zero



`move_joint(joint_id, position, move_time)`
{:.docsubitem#move_joint}

Move a specific joint, selected by `joint_id` (0 to 8) to `position` taking `move_time` milliseconds


`SIDE_CODES`
{:.docsubitem#SIDE_CODES}

A str, one of `'left'`, `'right'`, `'forward'` and `'backward'`.
These are relative to Marty's facing direction.


`lean(direction, amount, move_time)`
{:.docsubitem#lean}

Lean over in a direction, taken from `SIDE_CODES`


`walk(num_steps=2, start_foot='left', turn=0, step_length=25, move_time=1500)`
{:.docsubitem#walk}

Instructs the robot to start walking, with defaults set for all parameters.
`move_time` is in milliseconds (1/1000 of a second), `step_length` is *roughly* millimetres.


`eyes(angle)`
{:.docsubitem#eyes}

Move the eyes to `angle` position. 


`kick(side, twist, move_time)`
{:.docsubitem#kick}

Kick with the foot on `side` (again from `SIDE_CODES`) 


`arms(right_angle, left_angle, move_time)`
{:.docsubitem#arms}

Move the arms to each respective angle, taking `move_time` milliseconds


`celebrate(move_time=1000)`
{:.docsubitem#celebrate}

Do a little dance, with a given default movement time


`sidestep(side, steps, step_length, move_time)`
{:.docsubitem#sidestep}

Walk sideways to `side` (from `SIDE_CODES`) with *roughly* millimetre `step_length` taking `move_time`


`stand_straight(move_time=1000)`
{:.docsubitem#stand_straight}

Move the robot to the zero positions. Similar to `hello()` but doesn't wiggle the eyes


`play_sound(freq_start, freq_end, duration)`
{:.docsubitem#play_sound}

Play a tone that linearly interpolates between the Frequency `freq_start` in Hz to `freq_end`,
taking `duration` seconds to play.


`get_battery_voltage()`
{:.docsubitem#get_battery_voltage}

Returns a `float` that is the reading of the battery voltage from the board.


`get_accelerometer(axis)`
{:.docsubitem#get_accelerometer}

Returns a `float` reading of the `axis = 'x' or 'y' or 'z'` from the board. Note the axes
are marked on the control board, see [here](/hardware/sensors/) also for conventions and orientation.

The readings are in Gs, i.e. `1` represents an acceleration of 9.81ms<sup>-2</sup> in that direction.


`get_motor_current(motor_id)`
{:.docsubitem#get_motor_current}

Returns a `float` of the current detected on the `motor_id`'s channel by the control board



`digitalread_gpio(gpio)`
{:.docsubitem#digitalread_gpio}

Returns the `HIGH`/`LOW` state of a GPIO pin (0 to 8) as `True` or `False`


`enable_motors(enable=True)`
{:.docsubitem#enable_motors}

Toggles whether the motors are enabled (allowed to move) or disabled, where they can freely move.
This is called just before the `Marty` constructor completes.



`fall_protection(enable)`
{:.docsubitem#fall_protection}

`enable = True or False` boolean toggle for Fall Protection on the Control board, which automatically
disables motors when Marty falls over.



`motor_protection(enable)`
{:.docsubitem#motor_protection}

`enable = True or False` boolean toggle for Motor Protection on the Control board, which automatically
disables motors when the current readings pass a threshold. Disable this feature to prevent automatic
disabling.


**Be Aware** that the motor *protection* is there for a reason, to prevent you from breaking the motors.
It can be surprisingly easy to destroy a microservo motor by forcing it to move when it doesn't want to.
We won't cover warranty replacements for motors if you disable protections and then break a motor.
{:.alert.warning.tag}


`battery_protection(enable)`
{:.docsubitem#battery_protection}

`enable = True or False` boolean toggle for Battery Protection on the Control board, which automatically
disables motors and sounds an alarm when Marty's battery is drained. Disable this if you're using a
power supply that is safe to use at lower voltages.


**Never** discharge the supplied Lithium Polymer (LiPo) battery below **6.4 Volts** as this can
permanently damage the battery or even cause it to catch fire. You should stop using your Marty,
power it off and charge it if the battery voltage gets this low.
{:.alert.danger.tag}



`buzz_protection(enable)`
{:.docsubitem#buzz_protection}

`enable = True or False` boolean toggle for Buzz Protection on the Control board, which is a feature
that tries to reduce steady-state buzzing from the motors when Marty isn't moving.



`save_calibration()`
{:.docsubitem#save_calibration}

Saves the **current pose** of the Robot as the zero position

Your Robot may interfere with itself (robot speak for hit itself) if you calibrate it wrong.
This will probably result in a robot unable to walk properly until you recalibrate it.
{:.alert.warning.tag}



`ros_command(byte_array)`
{:.docsubitem#ros_command}

*(Generally Specific to the Socket-type client)*

Low level proxied access to the ROS Serial API between the modem and main controller.
Will forward the given `byte_array` (i.e. list of chars/bytes) through to the main controller.


`_pack_uint16(num)`
{:.docsubitem#pack_uint16}

Packs a (assumingly unsigned) 16-bit integer `num` into two 8-bit bytes,
returned as a tuple `(LSB, MSB)`





`martypy.MartyCommandException`
{:.docitem#MartyCommandException}

Raised if an argument given to a command wasn't recognised, for example:

```
mymarty.get_accelerometer('abc')
...
    290         except KeyError:
    291             raise MartyCommandException("Axis must be one of {}, not {}"
--> 292                                         "".format(self.ACCEL_AXES.keys(), axis))
    293         return self.client.execute('accel', ax)
    294 

MartyCommandException: Axis must be one of {'x', 'z', 'y'}, not 'abc'
```



`martypy.MartyConnectException`
{:.docitem#MartyConnectException}

Raised if the `Marty` class (and it's client) cannot connect to or locate the robot.


`martypy.ArgumentOutOfRangeException`
{:.docitem#ArgumentOutOfRangeException}

Raised when an argument that has constraints such as needing to fit within an 8-bit integer
has broken those constraints, e.g. by overflowing.


`martypy.UnavailableCommandException`
{:.docitem#UnavailableCommandException}

Raised when a command has been called that is not supported by the client type
specified in the `Marty` URL or elsewhere.


<a name="client-types"></a> Client Types & GenericClient
===

You can hook in to the `Marty` class and provide a new type of client that implements custom
handling of all the commands called by `Marty` or monkey-patch an
existing client type by passing a `client_types` dict to the `Marty(...)` constructor.



`martypy.GenericClient(*args, **kwargs)`
{:.docitem#GenericClient}

Provides a generic interface as expected by the `Marty` class. Child classes should
overwrite `__init__` with whatever initialisation code they need, and at this stage add
command handlers to the `COMMANDS_LUT` dict using `register_commands(...)` or otherwise.

The existing `execute` function within `GenericClient` does a lookup against `COMMANDS_LUT`
and calls the given handler, raising `martypy.UnavailableCommandException` if it hasn't been
set by this client, or `martypy.MartyCommandException` if any other exception occurred.


{% highlight python %}
import martypy

class MyClient(martypy.GenericClient):
    ...

mymarty = Marty('myclient://marty', client_types={'myclient' : MyClient})
{% endhighlight %}



`COMMANDS_LUT`
{:.docsubitem#COMMANDS_LUT}

Lookup table used to link together commands called my `Marty` with command handlers.

See the implementation of `martypy.socketclient.SocketClient.__init__` for more info.


`register_commands(handlers)`
{:.docsubitem#register_commands}

Takes a `dict` of `{ str : callable }` which are dict-merged in to `COMMANDS_LUT`, giving
preference to the newer handler in `handlers`


`execute(*args, **kwargs)`
{:.docsubitem#execute}

Calls a command, `args[0]` with the command handler against that key in `COMMANDS_LUT`,
passing `self`, `*args` and `**kwargs` to it. Since `self` is passed, non-class member
handlers might have some issues working correctly.




<br>
