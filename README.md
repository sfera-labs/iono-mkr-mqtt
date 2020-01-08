# Iono MKR MQTT

This sketch turns [Iono MKR](https://www.sferalabs.cc/iono-mkr/) into a LoRaWAN Class A/C end-device, using ABP activation and sending data encoded with the Cayenne LPP format.

This sketch requires the [Iono library](https://github.com/sfera-labs/iono/tree/master/Iono) to be installed and includes the [CayenneLPP library](https://github.com/sabas1080/CayenneLPP) and the (modified) [MKRWAN library](https://github.com/arduino-libraries/MKRWAN).

## Configuration

After uploading the sketch to Iono MKR, you'll be able to configure it via serial console.

The configuration console can be accessed through the module's RS-485 port or Arduino's USB port using any serial communication application (e.g. the Serial Monitor of the Arduino IDE).

Set the communication speed to 9600, 8 bits, no parity, no flow-control and connect the cable.

When the module is powered-up or reset, you can enter console mode by typing five or more consecutive space characters within 20 seconds from boot. The following will be printed:

```
=== Sfera Labs - Iono MKR LoRaWAN configuration - v1.0.0 ===

    1. Import configuration
    2. Export configuration

> 
```

Enter `2` to print out the current configuration. If the device is not configured, an example configuration will be printed:

```
> 2

*** Not configured ***

Example:

devAddr: 00000000
nwkSKey: 00000000000000000000000000000000
appSKey: 00000000000000000000000000000000
band: EU
data rate: 5
input modes: DDDDDD
I/O rules: ----
```

Copy the configuration lines to a text editor and edit the parameters:

- **devAddr**: 4-bytes device address in hexadecimal format
- **nwkSKey**: 16-bytes network key in hexadecimal format
- **appSKey**: 16-bytes application key in hexadecimal format
- **band**: regional band to use. Allowed values are: `AU`, `EU`, `KR`, `IN` and `US`
- **data rate**: `0` (SF 12, BW 125), `1` (SF 11, BW 125), `2` (SF 10, BW 125), `3` (SF 9, BW 125), `4` (SF 8, BW 125), or `5` (SF 7, BW 125)
- **inputs modes**: input mode of each of the 6 inputs: inputs 1 to 4 can be used as digital (`D`), voltage (`V`) or current (`I`); inputs 5 and 6 only as digital. If you do not want an input to trigger LoRaWAN updates, set it to "ignore" (`-`). E.g.: `D-VID-` means the following inputs will be monitored: DI1, AV3, AI4, and DI5
- **I/O rules**: linking rules between digital inputs and corresponding output relay (DI1 -> DO1 ... DI4 -> DO4). The possible rules are:
    - `F`: follow - the relay is closed when input is high and open when low    
    - `I`: invert - the relay is closed when input is low and open when high    
    - `H`: flip on L>H transition - the relay is flipped at any input transition from low to high    
    - `L`: flip on H>L transition - the relay is flipped at any input transition from high to low    
    - `T`: flip on any transition - the relay is flipped at any input transition, both high to low and low to high    
    - `-`: no rule - no control rule set for this relay.
    
After editing the configuration, enter `1` (Import configuration) in the serial console, copy all the lines together and paste them to the console:

```
> 1

Paste the configuration:

New configuration:

devAddr: 0011aabb
nwkSKey: 85f07ac17324760061e5d94dafcb7135
appSKey: ba069e01c4ca27535460a2b13ee6a515
band: EU
data rate: 5
input modes: D-VID-
I/O rules: FI--

Confirm? (Y/N):

> Y

Saving...
Saved!
Resetting... bye!
```

Review and confirm (enter `Y`) the configuration.  After saving, Iono MKR will automatically restart with the new configuration which is persisted in the Arduino's Flash memory and retained across restarts and power cycles.

## Data format

When configured, Iono MKR will start sending state uplinks periodically and upon state changes. The packet payload is encoded using the [Cayenne LPP format](https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload).

The full state (i.e. all enabled Cayenne channels, see below) is sent at start-up and then every 15 minutes or every 7 minutes if no other update is sent.

Partial updates are sent when monitored inputs or outputs change state. A 50ms debounce is used on digital and analog inputs and a min variation of 0.1V or 0.1mA is required on analog inputs to trigger an update.

Moreover, a heartbeat containing only channel 99 is sent every 3 minutes.

Downlink commands, encoded in Cayenne LLP format, can be sent at any time to change the state of the outputs (Cayenne channels 101 to 106 and 201).

### Cayenne channels

|Channels|Type|Description|
|:------|:--:|-----------|
|1 ... 6|Digital Input|DI1 ... DI6 state|
|11 ... 14|Analog Input|AV1 ... AV4 voltage (V)|
|21 ... 24|Analog Input|AI1 ... AI4 current (mA)|
|51 ... 56|Analog Input|DI1 ... DI6 counter, increased on every rising edge. Range: 0-327 (rolls back to 0 after 327)|
|101 ... 106|Digital Output|DO1 ... DO6 state|
|201|Analog Output|AO1 voltage (V)|
|99|N/A|2-bytes internal counter|
