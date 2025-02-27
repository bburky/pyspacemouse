# pyspacemouse

3Dconnexion Space Mouse in Python using raw HID.
Note: you **don't** need to install or use any of the drivers or 3Dconnexion software to use this package.
It interfaces with the controller directly with `hidapi` and python wrapper library `easyhid`.

Connected project [pySpaceApp](https://github.com/JakubAndrysek/pyspaceapp) is a simple example of how controll your PC
with SpaceMouse.

Forked from: [johnhw/pyspacenavigator](https://github.com/johnhw/pyspacenavigator)

Implements a simple interface to the 6 DoF 3Dconnexion [Space Mouse](https://3dconnexion.com/uk/spacemouse) device as
well as similar devices.

![](media/spacemouse-robot.jpg)
Control a [robot](https://roboruka.robotickytabor.cz/) with a Space Mouse

## Supported 3Dconnexion devices

* SpaceNavigator
* SpaceMouse Pro
* SpaceMouse Pro Wireless
* SpaceMouse Wireless
* 3Dconnexion Universal Receiver
* SpaceMouse Compact
* SpacePilot Pro
* [Add more devices](https://github.com/johnhw/pyspacenavigator/issues/1)

## Installation

- [hidapi](https://github.com/libusb/hidapi) is `C` library for direct communication with HID devices
    - ### Linux
        - [libhidapi-dev]() to access HID data
        - `sudo apt-get install libhidapi-dev` (Debian/Ubuntu)
        - Compile and install [hidapi](https://github.com/libusb/hidapi/#build-from-source).  (other Linux
          distributions)

        - add rules for permissions
            ```bash
            sudo echo 'KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0664", GROUP="plugdev"' > /etc/udev/rules.d/99-hidraw-permissions.rules
            sudo usermod -aG plugdev $USER
            newgrp plugdev
            ```

    - ### Windows
        - Install the latest release of hidapi.dll and hidapi.lib from
          the [hidapi releases](https://github.com/libusb/hidapi/releases) page.
        - Set system environment: add absolute path for `x64` or `x86` folder in Path.

    - ### Mac OS X
        - Install from [Homebrew](https://formulae.brew.sh/formula/hidapi)
        - `brew install hidapi`

- [easyhid](https://github.com/bglopez/python-easyhid) is `hidapi` interface for Python - required on all platforms
    - `pip install git+https://github.com/bglopez/python-easyhid.git`
    - this fork fix problems with `hidapi` on MacOS.
    - on other platforms it possible works with original package `pip install easyhid`

## Basic Usage:
[basicExample.py](examples/basicExample.py)
````py
import pyspacemouse
import time

success = pyspacemouse.open()
if success:
    while 1:
        state = pyspacemouse.read()
        print(state.x, state.y, state.z)
        time.sleep(0.01)
````

## State objects

State objects returned from read() have 7 attributes: [t,x,y,z,roll,pitch,yaw,button].

* t: timestamp in seconds since the script started.
* x,y,z: translations in the range [-1.0, 1.0]
* roll, pitch, yaw: rotations in the range [-1.0, 1.0].
* button: list of button states (0 or 1), in order specified in the device specifier

## Usage with callback
[callbackExample.py](examples/callbackExample.py)
````py
import pyspacemouse
import time


def button_0(state, buttons, pressed_buttons):
    print("Button:", pressed_buttons)


def button_0_1(state, buttons, pressed_buttons):
    print("Buttons:", pressed_buttons)


def someButton(state, buttons):
    print("Some button")


def callback():
    button_arr = [pyspacemouse.ButtonCallback(0, button_0),
                  pyspacemouse.ButtonCallback([1], lambda state, buttons, pressed_buttons: print("Button: 1")),
                  pyspacemouse.ButtonCallback([0, 1], button_0_1), ]

    success = pyspacemouse.open(dof_callback=pyspacemouse.print_state, button_callback=someButton,
                                button_callback_arr=button_arr)
    if success:
        while True:
            pyspacemouse.read()
            time.sleep(0.01)


if __name__ == '__main__':
    callback()
````

## API

    open(callback=None, button_callback=None, button_callback_arr=None, set_nonblocking_loop=True, device=None)
        Open a 3D space navigator device. Makes this device the current active device, which enables the module-level read() and close()
        calls. For multiple devices, use the read() and close() calls on the returned object instead, and don't use the module-level calls.

        Parameters:
            callback: If callback is provided, it is called on each HID update with a copy of the current state namedtuple
            dof_callback: If dof_callback is provided, it is called only on DOF state changes with the argument (state).
            button_callback: If button_callback is provided, it is called on each button push, with the arguments (state_tuple, button_state)
            device: name of device to open, as a string like "SpaceNavigator". Must be one of the values in `supported_devices`.
                    If `None`, chooses the first supported device found.
        Returns:
            Device object if the device was opened successfully
            None if the device could not be opened

    read()              Return a namedtuple giving the current device state (t,x,y,z,roll,pitch,yaw,button)
    close()             Close the connection to the current device, if it is open
    list_devices()      Return a list of supported devices found, or an empty list if none found

open() returns a DeviceSpec object. If you have multiple 3Dconnexion devices, you can use the object-oriented API to
access them individually.
Each object has the following API, which functions exactly as the above API, but on a per-device basis:

    dev.open()          Opens the connection (this is always called by the module-level open command,
                        so you should not need to use it unless you have called close())
    dev.read()          Return the state of the device as namedtuple [t,x,y,z,roll,pitch,yaw,button]
    dev.close()         Close this device

There are also attributes:

    dev.connected       True if the device is connected, False otherwise
    dev.state           Convenience property which returns the same value as read()

## Predefined callbacks

````py
import pyspacemouse
import time

success = pyspacemouse.open(dof_callback=pyspacemouse.print_state, button_callback=pyspacemouse.print_buttons)
if success:
    while 1:
        state = pyspacemouse.read()
        time.sleep(0.01)
````

### Callback: print_state

Print all axis states

    x +0.00    y +0.00    z +0.00 roll +0.00 pitch +0.00  yaw +0.00    t +0.0

### Callback: print_buttons

Print all buttons states

    [ 0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0, ]

## Troubleshooting

### ModuleNotFoundError: No module named 'easyhid'

- Install `easyhid` by `pip install easyhid`.

### AttributeError: function/symbol 'hid_enumerate' not found in library '<None>': python3: undefined symbol: hid_enumerate

- HID library for your computer is not installed.
- Follow the instructions in [requirements](#requirements).