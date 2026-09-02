# ModelTrainTurnoutControl

## Introduction
Code to control the position and polarity of turnouts on a model train layout. 
I am creating my own N scale layout with 18 (or more!) turnouts, and a Tortoise controller
costs $25+ US. There are lots of examples of using servos for turnout control out there,
and between using 3D printed mounts, and servos and controller board from AliExpress, creating
my own controller seems very attainable.

My goal is to have a control board of switches where
a single switch can control a single turnout or two turnouts paired together. The frog polarity
will also be controlled via relays, and should be compatible with DC or DCC (though I am aiming for
a DCC layout).

## Design 
The controller will read two configuration files at startup. The first file will represent
the physical representation of the relays, servos, turnouts and switches. This configuration file will
only change with physical changes to the track. The second file will represent the more dynamic configuration
of the turnouts, notable the positions of the turnout when in normal and reverse positions, and the expected
polarity of the frog when in the normal position (and thus the opposite polarity when in reverse position).
Using the values in these configuration files, turnouts will be set to the positions (and polarities) to match
the current physical positions of the switches. Subsequent flipping of switches will change position and polarity
of the connected turnout (or paired turnouts).

### Hardware
#### Microcontroller
I am going to use the Arduino architecture for the primary microcontroller architecture. There are plenty of
available designs to choose from, but the main limiting factor is the number of turnouts. Any reasonably large
layout is going to have a fair number of turnouts. My current layout is expecting 18 turnouts. With no turnout
pairing, that would require 18 individual switches to be monitored as well as 18 output pins for controlling relays
for frog polarity. That quickly exceeds the number of available pins on the basic Arduino Uno. One could look at
using the Arduino Mega instead, but I have never been a fan of the Mega. One can also look at using I2C to reduce
the number of pins required, especially for servo and relay control. I have decided to use the 
[Teensy 4.1 microcontroller](https://www.pjrc.com/store/teensy41.html). It has a fast, modern processor, lots of
on board memory, 55 digital input/output pins, a small physical footprint, and has a built-in SD card port. It can
even support Ethernet if that is desired in the future. It does require 3.3v for all of its pins, so that is a conderation
when evaluating other components. It should be plenty of power for this implementation, with
room to expand. Some design choices are going to be influenced by this architecture decision, so if you decide to
use a different microcontroller, changing the code will be an issue.

#### Servo
I have opted for the [MG90S 9g Servo](https://www.aliexpress.us/item/3256806032951610.html) with all metal gears.
The metal gears should provide better control and longevity of the servo. Each servo is just a couple of dollars and
is a very standard shape. It is commonly available from many different manufacturers and suppliers.

My current servo mount allows for the servo to be mounted vertically, and the movement of the arm attached to servo
will be about the 90 degree servo position.

#### Servo Controller
I am currently designing with the [16 Channel 12-bit PWM Servo Motor Driver LU9685 Driver servo controller](https://www.aliexpress.us/item/3256806206560666.html) in mind.
It uses I2C to control up to 16 servos per board, is very affordable, and there is an existing Arduino library available
on GitHub. The 5V used to control the servos can be isolated from the voltage used for the I2C
communication, so 3.3v can be used.

The other option is the [PCA9685 controller board](https://www.aliexpress.us/item/3256810335083711.html) which is also
very popular and comparable. Choosing one over the other should not affect the overall architecture as the controller
can be contained in the underlying code with the expected functional API to be the same which ever one is used. It should
not be difficult to replace the LU9685 implementation with a PCA9685 implementation.

#### Relay Controller
For controlling the frog polarity, I have opted to use Optocoupler relays instead of physical limit switches mounted with
the servo. There are some nice designs out there that will activate switches as the servo position is changed, and thus
change the polarity that is run through the switch. However, in running some tests on a prototype, the amount of position
change required for my N scale turnouts is actually very small. And I don't want to fiddle with the position of a switch
to make sure it is properly activated by the movement of the servo and how much it needs to move. Much simpler to just
control the polarity by flipping a relay one way or the other.

The relay controller I am designing for is the [XL9535 8 or 16 Channel Expansion Optocoupler Isolation Board](https://www.aliexpress.us/item/3256806206560666.html).
I chose this controller because it can handle up to 10A per relay, more than enough for most DCC setups, and certainly
enough for mine. It supports I2C communication, so multiple boards cane be used on a layout. It comes in 8 or 16 channels so you
can mix and match however many you may need. The 5V used to control the relays can be isolated from the voltage used for the I2C
communication, so 3.3v can be used.

### Software
The software to run the controller will be developed using the Arduino IDE. Development using built-in and support libraries
will be used. The main execution will be an Arduino sketch that is meant to be generally useful, but can be modified to match
a specific layout if need be. The main configuration of the software will be in the two configuration files mentioned earlier.

#### Libraries
These libraries will be required in the development and compilation of the controller software.

##### [ArduinoLogging](https://github.com/markwomack/ArduinoLogging/tree/main)
ArduinoLogging is used to log debugging information to serial for debugging purposes.

##### [TaskManager](https://github.com/markwomack/TaskManager/tree/main)
TaskManager is used as a lightweight task management system to control the various behaviors of the controller.

##### [LU9685](https://github.com/eleboys/LU9685/tree/main)
LU9685 is used to control the LU9685 servo controller.

##### [TCA9555](https://github.com/RobTillaart/TCA9555/tree/master)
TCA9555 is used to control the TCA9535 relay controller.

## Design considerations

### Independent of DCC
I am designing this controller to be independent of DCC and it will not act as a DCC device on the layout. Beyond the
dumb switching of frog polarity when flipping a turnout position, this controller will not be using or be available on
the DCC bus. I might revisit this in the future, but for now I am fine with it being a separate, self-contained system
on my layout.

### SD Card vs eeprom
I have reviewed some designs that used eeprom to store configuration data between uses. The data could be stored in the built-in
eeprom (of which the Teensy 4.1 has 4K available) or on a separate eeprom chip. I have decided to utilize the SD card functionality
of the Teensy 4.1. I think that being able to swap out the SD card and edit the configuration files directly will be an
advantage and will make usage of the controller much easier.
