# air_freshener_ESPhome

This project turns an Air Wick air-freshener into a motion triggered PIR device that communicates with Home Assistant via MQTT, using the Wemos D1 mini ESP8266 module.

The components required are:

1) Air Wick air-freshener 
2) Wemos D1 Mini (original/clone)
3) AM312 PIR module
4) CE012 DC Boost converter 
5) Schottky diode (nothing major, I had a 100mA available)
6) Extra wires
7) Soldering equipment

By referring to the included diagram, the ESP8266 is naturally in a disabled state. To achieve this, locate and remove the pull-up resistor for pin CH_EN of the ESP8266. In my case, that was the bottom-most resistor on the right side at the back of the Wemos module. 

Connect this pin to the D2 pin, which will serve as a "keep awake" pin, whilst the ESP8266 does all the required comms. 
It will also be connected via the schottky diode to the PIR sensor. This particular PIR sensor outputs HIGH on motion detection and keeps HIGH on re-triggering. The diode is required so that we can read the state of the sensor without reading the D2, which will be always high during normal operation.

We read the state of the PIR sensor by probing the connection between PIR and the diode using D1.

The module is powered by the batteries of the air wick unit. We pick up power using the "Batt+" and "Gnd" rails and feed that to the 3v3 DC boost converter. This converter will allow operation of the complete unit from 0.8 up to 3V. 

Finally, 2 extra connections are required to operate the spray functionality; spray enable (enable_controller in yaml) and spray fire (trigger_spray in yaml)
Both of these activate on LOW. Whilst the ESP8266 is disabled, the pins are kept in high impedance. Once awaken, the pins are set to HIGH on boot and then when the timing condition is met, are sequently set to LOW, before being set to HIGH again prior to disabling the ESP8266. 

The logic inside yaml is simple. ESPhome is used as the base API.
Once awaken, check in loop when the timing condition is met using the time from a local sntp server (running within HASS, or you could use any other)
As long as we are not in ota_mode (more on that later), save the time locally and spray. Wait until as set amount of time has passed (10s using the uptime sensor) and as long as there's no motion, disable the chip.

When it comes to sensors, there's an ADC (battery monitoring using A0), wifi signal, motion (binary)

The switches are set to internal, as there is no real point exposing them to HASS.

For the MQTT, there are 2 messages that are of interest
One that carries the ON/OFF payload for the ota_mode, which basically disables spraying and keeps the unit from disabling. You flick a switch in HASS and the next time it communicates with the MQTT server, it stays on. 
The other carries the latest timestamp using a json message. 


There are tons of improvements to be made, namely the use of an RTC module, using the onboard flash memory for storing the timestamp, setting more conditions to meet edge cases, optimizing wake on time etc.
