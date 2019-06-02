Documentation about the internal mechanics and functionalities of the sensor controller.

`Disclaimer`: All development and testing has been done using a **Raspberry Pi 3 Model B+**

## Software Components
Component | Description | Link
--- | --- | ---
GitHub Repo | Source code of sensor controller | <https://github.com/dangpg/BikeVR-Sensor-Controller>
Python BLE Library | GitHub repo to python library responsible for talking to the BlueZ stack | <https://github.com/ukBaz/python-bluezero>
cwiid | GitHub repo to python library which allows reading Wiimote internal values over Bluetooth | <https://github.com/abstrakraft/cwiid>

## Requirements
1. Run and tested only on Raspberry Pi 3 Model B+
2. Requires Python 2.7 (<https://www.python.org/download/releases/2.7/>)
3. Clone project and run `pip install -r requirements.txt`
4. In addition, run `sudo apt-get install python-cwiid`

## How to Run
Start sensor controller by running `python -m bikevr_sensor_controller`. The Raspberry Pi will be automatically advertise as a BLE peripheral. There are certain commands which allow for more options: (just enter these into the console)

1. `wii` - Starts scanning for a nearby Wiimote. Press 1+2 on the Wiimote to connect. When successfully connected, the respective characteristics will be updated.

2. `exit` or `quit` - Quits the application

## Sensor Controller Service
Name | UUID
--- | ---
Sensor-Controller Service | `00000000-0000-4b23-9358-f235b978d07c`

Randomly generated UUID. Can be edited within `app.py`. We decided to put every characteristic into one single service (makes it easier later on when connecting to the sensor-controller).

## IR Characteristic (TCRT5000)
Name | UUID | Format | Description 
--- | --- | --- | ---
IR Sensor | `11111111-1111-4b23-9358-f235b978d07c` | boolean | Represents state of TCRT5000 sensor. 1 indicates that infrared is getting reflected back to the phototransistor, i.e. something is in front of the sensor and blocking its way.

In order to connect the sensor to the Raspberry Pi, simply use three jumper wires to connect the respective pins:

TCRT5000 Pins | Raspberry Pi Pins
--- | ---
VCC | Pin #2 (DC Pover 5V)
GND | Pin #6 (Ground)
DO | Pin #7 (GPIO04)

When powering up the Raspberry Pi, the sensor should automatically turn on and indicating its opearting state by its LEDs.

## Wiimore Characterisitc
Name | UUID | Format | Description 
--- | --- | --- | ---
Wiimote Status | `22222222-2222-4b23-9358-f235b978d07c` | boolean | Indicates whether Wiimote is connected or not
Wiimote Buttons | `33333333-3333-4b23-9358-f235b978d07c` | unsigned int 16 | Represents the current state of the Wiimote buttons

Button mapping of Wiimote using python-cwiid when holding it vertically:

    1 - Button 2
    2 - Button 1
    4 - Button B
    8 - Button A
    16 - Minus
    32 - 
    64 -
    128 - Home
    256 - Left
    512 - Right
    1024 - Down
    2048 - Up
    4096 - Plus

When holding multiple buttons down, their respective values get summed up.

## Write Own Characteristic

In order to write your own characteristic / add an additional sensor, one can use the following template to advertise it as a BLE characteristic:

    from bikevr_sensor_controller.common.user_descriptor import UserDescriptor
    from bikevr_sensor_controller.common.format_descriptor import FormatDescriptor
    from bikevr_sensor_controller.common.tools import sint16, to_int
    from bluezero import peripheral
    from threading import Thread
    from time import sleep

    NAME = 'Name of Characteristic'

    POLL_RATE = 0.1 #seconds

    class TemplateCharacteristic(peripheral.Characteristic):    
        def __init__(self, uuid, service):
            peripheral.Characteristic.__init__(self,
                                    uuid,
                                    ['read', 'notify'],
                                    service,
                                    sint16(0))  # starting value
            self.add_descriptor(UserDescriptor(NAME, self))
            self.add_descriptor(FormatDescriptor([0x0E, 0x00, 0x00, 0x27, 0x01, 0x00, 0x00], self))

            self.add_notify_event(self.notify_event)
            

        def ReadValue(self, options):
            return self.value

        def notify_event(self):
            if self.notifying:
                self.thread = Thread(target = self.run, args = ())
                self.thread.setDaemon(True)
                self.thread.start()
            else:
                self.thread.join()

        def run(self):
            while self.notifying:
                # 1. Get new value
                # 2. Check whether value has changed
                # 3. Use self.send_notify_event to propagrate changes
                
                sleep(POLL_RATE)


1. Give your characteristic a name (simply edit the NAME variable)
2. If needed, adjust the poll rate (the interval a new value should be requested at)
3. Decide what format your advertised value should use (see <https://www.bluetooth.com/wp-content/uploads/Sitecore-Media-Library/Gatt/Xml/Descriptors/org.bluetooth.descriptor.gatt.characteristic_presentation_format.xml> for more information and possible values)
    1. Adjust the byte array parameter of FormatDescriptor based on your selection
    2. If needed, write converters for your format (see `tools.py` for an example)
4. Define a fitting starting value for your characteristic
5. Implement your own logic for calculating/retrieving new values (see the other characteristic for more examples)
6. Add your newly written characteristic to `app.py` so the sensor-controller includes it in its list
    1. Define a unique characteristic UUID
    2. Use `service.add_characteristic` to add your characteristic