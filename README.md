# Iono MKR MQTT

This sketch turns [Iono MKR](https://www.sferalabs.cc/iono-mkr/) into a client MQTT.

This sketch requires the [Iono library](https://github.com/sfera-labs/iono/tree/master/Iono) to be installed. It also uses ArduinoMqttClient v0.1.4., WiFiNINA v1.4.0 and FlashStorage v0.7.1.

## Configuration

After uploading the sketch to Iono MKR, you'll be able to configure it via serial console.

The configuration console can be accessed through the module's RS-485 port or Arduino's USB port using any serial communication application (e.g. the Serial Monitor of the Arduino IDE).

Set the communication speed to 9600, 8 bits, no parity, no flow-control and connect the cable.

When the module is powered-up or reset, you can enter console mode by typing five or more consecutive space characters within 10 seconds from boot. If the device is not configured you have no time limit to type space characters, if the device is already configured and you don't enter in console mode it will wait 10 seconds to start connecting with the network. The following will be printed:

```
=== Sfera Labs - Iono MKR MQTT configuration ===

    1. Import configuration
    2. Export configuration

> 
```

Enter `2` to print out the current configuration. If the device is not configured, an example configuration will be printed:

```
> 2

*** Not configured ***

Example:

netssid: networkname
netpass: networkpassword
brokeraddr: 192.168.1.128
brokerport: 1883
modes: DDDDDD
rules: F--I
keepalive: 120
qos: 1
retain: F
willtopic: will
willpayload: goodbye
clientId: sensor1
username: user
password: pass
roottopic: /iono-mkr/
```

Copy the configuration lines to a text editor and edit the parameters:

- **netssid**: ssid of the network you want to connect (max 100 chars)
- **netpass**: password of the network you want to connect (max 100 chars)
- **brokeraddr**: ip address of the MQTT broker you want to connect
- **brokerport**: port of the MQTT broker you want to connect
- **modes**: input mode of each of the 6 inputs: inputs 1 to 4 can be used as digital (`D`), voltage (`V`) or current (`I`); inputs 5 and 6 only as digital. If you do not want an input to trigger MQTT updates, set it to "ignore" (`-`). E.g.: `D-VID-` means the following inputs will be monitored: DI1, AV3, AI4, and DI5
- **rules**: linking rules between digital inputs and corresponding output relay (DI1 -> DO1 ... DI4 -> DO4). The possible rules are:
    - `F`: follow - the relay is closed when input is high and open when low    
    - `I`: invert - the relay is closed when input is low and open when high    
    - `H`: flip on L>H transition - the relay is flipped at any input transition from low to high    
    - `L`: flip on H>L transition - the relay is flipped at any input transition from high to low    
    - `T`: flip on any transition - the relay is flipped at any input transition, both high to low and low to high    
    - `-`: no rule - no control rule set for this relay
- **keepalive**: keep-alive timer (in seconds). This timer is used by the device to send a ping request (if no other messages are sent) to the MQTT broker to keep connection alive
- **qos**: quality-of-service of MQTT messages (publish messages and will message) sent by the device to the MQTT broker. Possible values: `0`,`1`,`2`
- **retain**: retain flag, if set to `T` every MQTT publish message will be stored by the MQTT broker. Possible values: `T`,`F`
- **willtopic**: topic of the MQTT will message used in case of accidental disconnection (max 100 chars). This field is not necessary for connection with the MQTT broker, if you don't want to use it remove this line entirely (don't leave this field empty)
- **willpayload**: payload of the MQTT will message used in case of accidental disconnection (max 100 chars). This field is not necessary for connection with the MQTT broker, if you don't want to use it remove this line entirely (don't leave this field empty)
- **clientId**: ID used for connection with the MQTT broker (max 100 chars)
- **username**: username used for connection authorization with the MQTT broker (max 100 chars)
- **password**: password used for connection authorization with the MQTT broker (max 100 chars)
- **roottopic**: root topic used as prefix to every MQTT publish topic and will topic. E.g.: if roottopic is '/iono-mkr/' and the device send a MQTT publish message on topic 'di1/val' (digital input 1) the whole topic will be '/iono-mkr/di1/val'. This field is not necessary for connection with the MQTT broker, if you don't want to use it remove this line entirely (don't leave this field empty)
 
NOTE: the MQTT will message will be stored by the MQTT broker only if both **willtopic** field and **willpayload** field are set. If **roottopic** field is not set the default root topic in every publish message and will message will be '/'

After editing the configuration, enter `1` (Import configuration) in the serial console, copy all the lines together and paste them to the console:

```
> 1

Paste the configuration:

New configuration:

netssid: networkname
netpass: networkpassword
brokeraddr: 192.168.1.128
brokerport: 1883
modes: DDDDDD
rules: F--I
keepalive: 120
qos: 1
retain: F
willtopic: will
willpayload: goodbye
clientId: sensor1
username: user
password: pass
roottopic: /iono-mkr/

Confirm? (Y/N):

> Y

Saving...
Saved!
Resetting... bye!
```

Review and confirm (enter `Y`) the configuration.  After saving, Iono MKR will automatically restart with the new configuration which is persisted in the Arduino's Flash memory and retained across restarts and power cycles.

## Data format

When configured, Iono MKR will start sending I/O state periodically and upon state changes.

The full state (i.e. all enabled I/O channels) is sent at start-up and then every 15 minutes or every 7 minutes if no other update is sent.

Partial updates are sent when monitored inputs or outputs change state. A 25ms debounce is used on digital and analog inputs and a min variation of 0.1V or 0.1mA is required on analog inputs to trigger an update.

Commands, encoded as MQTT publish messages, can be received at any time by the device to change the state of the outputs.

### I/O list with their topic

The specified topics are used by the device to send/receive MQTT publish messages

|I/O Type|Description|Topic|Read/Write|
|:------|:--:|-----------|:-:|
|Digital Input|DI1 ... DI6 [0/1]|roottopic + ('di1/val' ... 'di6/val')|R|
|Analog Input|AV1 ... AV4 voltage (mV)|roottopic + ('av1/val' ... 'av4/val')|R|
|Analog Input|AI1 ... AI4 current (µA)|roottopic + ('ai1/val' ... 'ai4/val')|R|
|Counter >= 0|DI1 ... DI6 counter, increased on every rising edge|roottopic + ('di1/count' ... 'di6/count')|R|
|Digital Output|DO1 ... DO4 [0/1]|roottopic + ('do1/val' ... 'do4/val')|R/W|
|Analog Output|AO1 voltage (mV)|roottopic + 'ao1/val'|R/W|

Input (digital and analog) are marked as 'R' as they can only be read. Outputs (digital and analog) are marked as 'R/W' as they can both be read and written. In order to change the state of an output the device has to receive a MQTT publish message on the specified topic of the output.

NOTE: 'roottopic' corresponds to the **roottopic** field defined in device configuration

Example (Input): if roottopic is '/iono-mkr/' and there's a variation on digital input DI1 then the whole topic of the corresponding MQTT publish message genereted will be '/iono-mkr/di1/val'

Example (Output): if roottopic is '/iono-mkr/' and the device want to receive a command on digital output DO1 than the whole topic of the corresponding MQTT publish message containing the command must be '/iono-mkr/do1/val'
