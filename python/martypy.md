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
{:.alert.info.tag}


Basic Usage
===

The `martypy` library has a core client class, `Marty` that provides
all the methods used to control a robot. The class takes one required argument,
which is a URL-like string that lets it know where to find your Marty.

{% highlight python %}
>>> from martypy import Marty
>>> mymarty = Marty('socket://192.168.0.2') # Change IP accordingly
>>> mymarty.hello()  # Move to zero positions and wink
True
>>> mymarty.play_sound(8000, 200, 1000) # Play a tone
>>> mymarty.walk() # Walk
{% endhighlight %}

If you're using the Marty remotely, over WiFi, that'll be `socket://` followed by the **IP Address**
of your robot, or a resolvable name, like `socket://192.168.0.42` or `socket://marty.local`. If your
robot can't be found, this step may throw an Exception.

<br>



A Note on Types
===

When referring to arguments that can be passed into the MartyPy methods, we'll give a C type
like `int8` or `uint16`. These define the acceptable range for the command (so for `int8`, -128 to 127,
and for `uint16`, 0 to 65,535.) Any values outside these will raise a Type or encoding exception.

Note that just because a command *can* take 65,535 as an argument that that is not necessarily then
a good idea nor something Marty can actually do. Experiment and play around!

Lastly, when a method takes `move_time`, this is always a `uint16` representing the number of
milliseconds (1/1000 of a second) that the commend should take. The nominal value for each movement
will be slightly different, and most of them have default arguments that seem good starting points
based on our experimentation with Marty.

<br>



<a name="main-members"></a> Main Functions and Members
===


None of the movement commands are *blocking*, meaning that if you tell the robot to
walk for two seconds, your program will keep running while that's happening rather than wait for
the walk action to finish. This can have both useful and undesireable side effects, so just
bear this in mind. Secondly, Marty *queues* actions, so if you send a walk, kick and then another
walk they will all happen one-agter the other, *not* at the same time or overwriting each-other.
{:.feature}


`martypy.Marty(url='socket://192.168.0._', client_types=dict(), *args, **kwargs)`
{:.docitem}

Class constructor for a Marty client instance, with a default URL given.

When you create a `Marty` instance, the `enable_safeties(True)` and `enable_motors(True)`
commands are sent to the Robot. If the kwarg `default_lifelike` is True, lifelike behaviours will also
be enabled.

`*args` and `**kwargs` are passed on to the client type, which will be chosen depending on the protocol
specified the URL. Currently the supported client types are `socket` and `test`.

For more info on extending the available client types via `client_types` see [here](#client-types).



`hello()`
{:.docsubitem#hello}

Moves to the zero position and wiggles the eyebrows. Be careful with this, as if the Robot doesn't
know where it it, it will move as quickly as it can to the zero positions, which can knock the
robot over. It's best used as the first command you send when you turn the Robot on, where it
should be close to the zero pose.


`discover()`
{:.docsubitem#discover}

Look for Martys available over whatever interface you're using, e.g. the `socket` client
will look for Martyies over the LAN. Return types from this command can be varied, depending
on the client interface. Some interfaces may not support this method.

For instance, with the Socket client type we of course can't create a `Marty` instance before
we know what it's IP address is. So, the `discover()` method is built into the Client itself,
and needs to be called like so:

{% highlight python %}
import martypy
martys = martypy.SocketClient.discover(timeout=10)
{% endhighlight %}

See also [the Socket API docs](/hardware/esp-socket-api/) for more information on the limits of discovery, and also
the [Simple Discovery Tool](/hardware/esp-socket-discovery/) which does a similar thing in JavaScript.



`stop(stop_type=None)`
{:.docsubitem#stop}

Stop the robot moving. `stop_type` is a str which should be a key in the `Marty.STOP_TYPE` dict.
If it is none (the default) then `'clear and stop'` will be assumed. Other options are:

clear queue
: clear movement queue only (so finish the current movement)

clear and stop
: clear movement queue and servo queues (freeze in-place)

clear and disable
: clear everything and disable motors

clear and zero
: clear everything, and make robot return to zero

pause
: pause, but keep servo and movequeue intact and motors enabled

pause and disable
: as above, but disable motors too



`move_joint(joint_id, position, move_time)`
{:.docsubitem#move_joint}

Move a specific joint, selected by `joint_id` (0 to 8) to `position` (-128 to 127) taking
`move_time` milliseconds



`lean(direction, amount, move_time)`
{:.docsubitem#lean}

Lean over in a direction, taken from `SIDE_CODES` taking `move_time` milliseconds


`walk(num_steps=2, start_foot='left', turn=0, step_length=25, move_time=1500)`
{:.docsubitem#walk}

Instructs the robot to start walking, with defaults set for all parameters.
`move_time` is in milliseconds (1/1000 of a second), `step_length` is *roughly* millimetres.
`turn` is an int8 in the range -128 to 127. 0 makes the robot walk straight ahead, negatives
move to the Robot's *right*, positives to the *left* (i.e. positive Y-Axis direction).


`eyes(angle, move_time=100)`
{:.docsubitem#eyes}

Move the eyes to `angle` position, taking `move_time` milliseconds


`kick(side='right', twist=0, move_time=2000)`
{:.docsubitem#kick}

Kick with the foot on `side` (again from `SIDE_CODES`) taking `move_time` milliseconds.
The `twist` (-128 to 127) argument adds a knee twist to the kick. 0 is straight ahead.


`arms(left_angle, right_angle, move_time)`
{:.docsubitem#arms}

Move the arms to each respective angle, taking `move_time` milliseconds


`celebrate(move_time=4000)`
{:.docsubitem#celebrate}

Do a little celebration, taking `move_time` milliseconds. THe default is sensible, though this
is hilarious/adorable with a move time around 1000.


`circle_dance(side, move_time)`
{:.docsubitem#circle_dance}

Makes Marty do a little dance in a circular motion. `side` should be a str from `SIDE_CODES`.


`sidestep(side, steps, step_length, move_time)`
{:.docsubitem#sidestep}

Walk sideways to `side` (from `SIDE_CODES`) with *roughly* millimetre `step_length` taking `move_time`


`play_sound(freq_start, freq_end, duration)`
{:.docsubitem#play_sound}

Play a tone that linearly interpolates between the Frequency `freq_start` in Hz to `freq_end`,
taking `duration` milliseconds to play.


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



`pinmode_gpio(gpio, mode)`
{:.docsubitem#pinmode_gpio}

Configure a GPIO pin's function. `mode` should be a str from `GPIO_PIN_MODES`.


`digitalread_gpio(gpio)`
{:.docsubitem#digitalread_gpio}

Returns the `HIGH`/`LOW` state of a GPIO pin (0 to 8) as `True` or `False`


`write_gpio(gpio, value)`
{:.docsubitem#write_gpio}

Write a value to a GPIO port. Acceptable value types depend on the GPIO configuration.


`enable_motors(enable=True, clear_queue=True)`
{:.docsubitem#enable_motors}

Toggles whether the motors are enabled (allowed to move) or disabled, where they can freely move.
This is called just before the `Marty` constructor completes.

Enable motors also *unpauses* the movement queue (see [stop](#stop)) so you probably also want to send
`stop('clear queue')` before enabling motors to prevent a jump. This is done by default with the
`clear_queue` argument.



`lifelike_behaviour(enable=True)`
{:.docsubitem#lifelike_behaviour}

If enabled, Marty will perform a short action every minute or so to remind you that it's on.
This is disabled by default, but we'd recommend turning it on.

If Lifelike Behaviours are on, they will **still** happen even if motors are disabled. To prevent
them from happening you need to explicitly disable Lifelike Behaviours.
{:.alert.tag.warning}


`enable_safeties(enable=True)`
{:.docsubitem#enable_safeties}

Enables all of the safety functions (battery voltage, fall, motor current) as well as raising
the battery voltage alarm so it'll go off a bit sooner. This is enabled by default by this library,
but is **not** on by default within firmware or potentially other APIs.

Use this as a quick way to enable all the below listed safety features to prevent you from breaking
your Marty by overstressing a Motor or your Battery
{:.alert.success.tag}


`fall_protection(enable=True)`
{:.docsubitem#fall_protection}

`enable = True or False` boolean toggle for Fall Protection on the Control board, which automatically
disables motors when Marty falls over. We'd strongly recommend turn leave this on.



`motor_protection(enable=True)`
{:.docsubitem#motor_protection}

`enable = True or False` boolean toggle for Motor Protection on the Control board, which automatically
disables motors when the current readings pass a threshold. Disable this feature to prevent automatic
disabling.


**Be Aware** that the motor *protection* is there for a reason, to prevent you from breaking the motors.
It can be surprisingly easy to destroy a microservo motor by forcing it to move when it doesn't want to.
We won't cover warranty replacements for motors if you disable protections and then break a motor.
{:.alert.warning.tag}


`battery_protection(enable=True)`
{:.docsubitem#battery_protection}

`enable = True or False` boolean toggle for Battery Protection on the Control board, which automatically
disables motors and sounds an alarm when Marty's battery is drained. Disable this if you're using a
power supply that is safe to use at lower voltages.


**Never** discharge the supplied Lithium Polymer (LiPo) battery below **6.4 Volts** as this can
permanently damage the battery or even cause it to catch fire. You should stop using your Marty,
power it off and charge it if the battery voltage gets this low.
{:.alert.danger.tag}



`buzz_protection(enable=True)`
{:.docsubitem#buzz_protection}

`enable = True or False` boolean toggle for Buzz Protection on the Control board, which is a feature
that tries to reduce steady-state buzzing from the motors when Marty isn't moving. This also
makes the Robot *compliant*, so you can gently move the robot's joints (apart from the eyes)
into a pose.



`save_calibration()`
{:.docsubitem#save_calibration}

Saves the **current pose** of the Robot as the zero position

Your Robot may interfere with itself (robot speak for hit itself) if you calibrate it wrong.
This will probably result in a robot unable to walk properly until you recalibrate it.
{:.alert.warning.tag}

**ProTip:** If you use `buzz_prevention(True)` you'll be able to slowly nudge the motors into
the correct positions (apart from the eyes), and then save that as the calibration for the Robot.
{:.alert.success.tag}


`ros_command(byte_array)`
{:.docsubitem#ros_command}

*(Generally Specific to the Socket-type client)*

Low level proxied access to the ROS Serial API between the modem and main controller.
Will forward the given `byte_array` (i.e. list of chars/bytes) through to the main controller.




`SIDE_CODES`
{:.docsubitem#SIDE_CODES}

A str, one of `'left'`, `'right'`, `'forward'` and `'backward'`.
These are relative to Marty's facing direction.


`STOP_TYPE`
{:.docsubitem#STOP_TYPE}

A str, one of `'clear queue'`, `'clear and stop'`, `'clear and disable'`,
`'clear and zero'`, `'pause'` and `'pause and disable'`


`GPIO_PIN_MODES`
{:.docsubitem#STOP_TYPE}

A str, one of `'digital in'`, `'analog in'` and `'digital out'`.


`_pack_uint16(num)`
{:.docsubitem#pack_uint16}

Packs an unsigned 16-bit integer `num` into two 8-bit bytes,
returned as a tuple `(LSB, MSB)`


`_pack_int16(num)`
{:.docsubitem#pack_int16}

Packs a signed 16-bit integer `num` into two 8-bit bytes,
returned as a tuple `(LSB, MSB)`


`_pack_uint8(num)`
{:.docsubitem#pack_uint8}

Packs an unsigned 8-bit integer `num` into one 8-bit byte


`_pack_int8(num)`
{:.docsubitem#pack_int8}

Packs a signed 8-bit integer `num` into one 8-bit byte


`_pack_float(num)`
{:.docsubitem#pack_float}

Packs a float `num` into four 8-bit bytes, returned as a tuple from
least significant byte to most.



`_mute_serial()`
{:.docsubitem#mute_serial}

This instructs the STM32 microcontroller on the board to stop monitoring the Serial
line between it and the ESP8266. THis is basically only useful for reprogramming
the ESP or very specific applications, such as shutting it up if you're using the 
serial line for something else. Basically, we doubt you'll need this command.



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


`martypy.MartyConfigException`
{:.docitem#MartyConfigException}

Raised if the `Marty` class (and/or it's client) have a configuration issue.


`martypy.ArgumentOutOfRangeException`
{:.docitem#ArgumentOutOfRangeException}

Raised when an argument that has constraints such as needing to fit within an 8-bit integer
has broken those constraints, e.g. by overflowing.


`martypy.UnavailableCommandException`
{:.docitem#UnavailableCommandException}

Raised when a command has been called that is not supported by the client type
specified in the `Marty` URL or elsewhere.








<a name="adding-commands"></a> Adding Commands or Clients
===

Execution of a command is resolved in MartyPy as follows:

The `Marty` class calls
the Client's method `execute(*args, **kwargs)`, with the command name as the first argument,
and any further arguments as necessary.

`execute` is implemented in `martypy.GenericClient`, which will perform a lookup against
the `GenericClient.COMMANDS_LUT` lookup-table. All the known commands shoulf be in there,
as `None`s, given the generic client just provides interface, not implementation.

If a callback is left as `None` then a `UnavailableCommandException` is raised when `Marty` tries
to call it - this is the preferred behaviour for client types that *don't* support a given command.

Each Client Type should subclass `GenericClient` and then register its own handlers for commands,
using the `register_commands` method, which basically merges the default `COMMANDS_LUT` dict with
any supplied dicts. The client class, say `SocketClient`, for example, would provide an appropriate
handler for each method. It's probably best to look at the source `martypy/socketclient.py` for
how exactly to do this.







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


The `SocketClient` is the main client type used to interface with Marty over WiFi, but the 
`TestClient` is useful for debugging or developing without having to have a Robot in the loop.

