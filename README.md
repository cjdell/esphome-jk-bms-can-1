# JK-BMS-CAN

[![Badge License: GPLv3](https://img.shields.io/badge/License-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Badge Version](https://img.shields.io/github/v/release/Sleeper85/esphome-jk-bms-can?include_prereleases&color=yellow&logo=DocuSign&logoColor=white)](https://github.com/Sleeper85/esphome-jk-bms-can/releases/latest)
![GitHub stars](https://img.shields.io/github/stars/Sleeper85/esphome-jk-bms-can)
![GitHub forks](https://img.shields.io/github/forks/Sleeper85/esphome-jk-bms-can)
![GitHub watchers](https://img.shields.io/github/watchers/Sleeper85/esphome-jk-bms-can)

**ESPHome component to monitor a Jikong Battery Management System (JK-BMS) and communicate with inverters supporting CAN bus protocol compatible with Pylontech, GoodWe, Seplos, SMA or Victron**

**Note: Pylontech uses 15s/48v and many others uses 16s/51.2v @3.2v/cell nominal.**
Other battery profiles that utilise the **PYLON** protocol with different cell counts may also work, eg Alpha Ess Smile, BYD Battery-Box LV Flex Lite.
Select the correct battery profile in the inverter to match your battery pack!

The ESP32 communicates with the JK-BMS using the RS485 port (GPS) which is in fact not RS485 but 3.3V UART-TTL, so it can be directly connected to the ESP32.
The ESP32 then sends the required CAN bus data to the inverter via a TJA1050, TJA1051T or SN65HVD230 CAN bus transceiver.

**Note: some inverters only accept a CAN bus at 3.3V in this case please choose the SN65HVD230 chip.**

**Sends over CAN bus to inverter:**
  - Battery Voltage
  - Battery Current (+charge, -discharge)
  - State of Charge (SOC)
  - State of health (SOH)
  - BMS Temperature
  - Charging Voltage
  - Charging Amps
  - Discharge min Voltage
  - Battery name
  - Alarms: Cell over/under voltage, Charge/discharge over current, High/low Temp, BMS fault
  
**Note: This code support only one BMS connection per inverter and should work with inverters that support the CAN bus protocol shown in the table below.
I'm only testing it with my Deye SUN-6K-SG03-LP1-EU inverter.<br>
The CAN bus support is still in development and testing...<br>**

## Charging Logic Diagram

The charging voltage and current correspond to the default values in the YAML script and can be modified.

![Image](images/JK-BMS-CAN_Charging_Logic_Diagram.png "JK-BMS-CAN Charging Logic")

## CAN Protocol Table

![Image](images/CAN_Protocol_Table.png "CAN Protocol Table")

Note: the following IDs are not implemented: 0x35B, 0x360, 0x372, 0x374, 0x375, 0x376, 0x377, 0x378, 0x380, 0x381 and 0x382.

## CAN Protocol Settings

This can be configured in the YAML file. It is configured by default for PYLON 1.2 protocol IDs (the most common protocol).

```YAML
# +--------------------------------------+
# | CAN Protocol Settings                |
# +--------------------------------------+
# CAN BMS Name (0x35E) : 0 NoSent / 1 PYLON / 2 GOODWE / 3 SEPLOS
  can_bms_name: "1"
# CAN alarm : 0 NoSent / 1 CanId 0x359 / 2 CanId 0x35A
  can_alarm: "1"
# CAN cell info : 0 NoSent / 1 CanId 0x70 / 2 CanId 0x373
  can_cell_info: "0"
# CAN battery info : 0 NoSent / 1 CanId 0x35F / 2 CanId 0x379
  can_battery_info: "0"
```

## Home Assistant and API

![Image](images/HA_Dashboard.png "HA Dashboard")

If your ESP32 is not connected with Home Assistant it will reboot every 15 minutes.
This is the normal behavior of ESPHome if HA is not connected to the ESP32 API.
This is not a bug to be resolved but a mechanism put in place by the ESPHome team to correct a possible problem with the API connection.

If you don't want to use Home Assistant, add the **"reboot_timeout: 0s"** option below the **"api:"** section of your YAML file.<br>
This option will disable the reboot every 15min.

```YAML
# +------------------------------------------------------------------+
# | ** The settings below can be modified according to your needs ** |
# +------------------------------------------------------------------+

api:
  reboot_timeout: 0s
```

## Web Server

If you don't use Home Assistant, you can activate the web server and have access to information coming from JK-BMS and also interact with the application.

![Image](images/ESPHome_Web_Server.png "Web Server")

The web server is not enabled by default.<br>
To enable this feature, uncomment the lines below in your YAML file.

```ỲAML
# +------------------------------------------------------------------+
# | ** The settings below can be modified according to your needs ** |
# +------------------------------------------------------------------+

web_server:
  port: 80
  log: false
  ota: false
```

## WiFi

By default, if the ESP32 is disconnected from the WiFi network it will reboot every 15 minutes to try to resolve the problem.<br>
If you don't want to connect the ESP32 to the WiFi network please remove the lines below in your YAML file.

```YAML
# +------------------------------------------------------------------+
# | ** The settings below can be modified according to your needs ** |
# +------------------------------------------------------------------+

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  domain: !secret domain
```

## Tips for Deye inverter

Add 0.1v to the voltage settings because the Deye charging voltage is always 0.1v lower than requested.
* Float V. : 53.7v (3.35v/cell - Natural voltage of a fully charged cell at rest, I advise you not to go higher.)
* Absorption V : 55.3v (3.45v/cell - It's not necessary to use a charging voltage higher than 55.2V for a full charge.)
* Absorption Offset V. : 0.15v (The absorption phase will start at 55.15v (BMS voltage). Warning: the BMS voltage must be correctly calibrated.)

## Changelog

* V1.15.4 Sleeper85 : Improved documentation for API, Web Server and WiFi settings
* V1.15.3 Sleeper85 : Add 'CAN Protocol Settings' and new CAN ID based on the SMA and Victron protocol (alpha)
* V1.15.2 Sleeper85 : Improved Alarm handling, all alarms will set charge/discharge current to 0A and set 'Charging Status' to Alarm
* V1.15.1 Sleeper85 : New CANBUS script with CANBUS Status in HA, stop sending CAN messages if the inverter is not responding (fix WDT reboot issues).
* V1.14.3 Sleeper85 : Improved documentation + Charging Voltage tips for Deye
* V1.14.2 Sleeper85 : Improve 'Charging Voltage' behavior
* V1.14.1 Sleeper85 : Add 'Float charge function'
* V1.13.6 Sleeper85 : Add 'Absorption time' and 'Absorption Offset V.' slider
* V1.13.5 Sleeper85 : Set CAN manufacter to "PYLON" for improve compatibility with Deye and other inverters
* V1.13.4 Sleeper85 : Improve 'Charge Status' behavior + add 'Rebulk Offset V.' slider
* V1.13.3 uksa007   : Improve compatibility with Deye and other inverters
* V1.13.2 uksa007   : Send Max Temperature of T1, T2 to inverter
* V1.13.1 uksa007   : Fix compile issues with new version of ESPhome 2023.4.0, set rebulk offset to 2.5

## Supported devices

Inverters supporting CAN Pylon/Goodwe Low Voltage protocol should work, check your inverter manual to confirm.

The following are confirmed and known to work:
* Deye SUN-6k-SG03LP1-EU (reported by [@Sleeper85](https://github.com/Sleeper85/esphome-jk-bms-can) using the battery mode 'Lithium 00')
* Deye SUN-5k-SG03LP1-EU (reported by [@vdiex](https://github.com/Uksa007/esphome-jk-bms-can/discussions/1#discussioncomment-4481364))
* Deye SUN-12K-SG04LP3-EU (reported by [@lucize](https://github.com/Uksa007/esphome-jk-bms-can/discussions/25#discussioncomment-5890844))
* Goodwe 3648-ES (GW5048-ES) (reported by [@jirdol](https://github.com/Uksa007/esphome-jk-bms-can/discussions/1#discussioncomment-5498743))
* Goodwe GW5000S-BP (reported by [@Uksa007](https://github.com/Uksa007/esphome-jk-bms-can/discussions/2#discussion-4469605) using the "Goodwe LX U5.4-L * 3" battery profile)
* Goodwe GW5000S-BP & GW3600S-BP (reported by [@OselDusan7](https://github.com/Sleeper85/esphome-jk-bms-can/discussions/4#discussion-6022729))
* Sofar ME 3000-SP (reported by [@starman](https://diysolarforum.com/threads/jk-bms-can-bus-comms-now-possible-for-inverters-that-support-goodwe-and-pylontech-batteries.48963/post-755539))
* Turbo Energy (reported by [@ibikku](https://github.com/Uksa007/esphome-jk-bms-can/discussions/13#discussion-4823950))
* Growatt SPF 5000ES (reported by [@Paulfrench35](https://diysolarforum.com/threads/jk-bms-can-bus-comms-now-possible-for-inverters-that-support-goodwe-and-pylontech-batteries.48963/page-21#post-965233) using L52 CAN protocol)


<br>All JK-BMS models with software version `>=6.0` are using the implemented protocol and should be supported.

See the [@syssi](https://github.com/syssi) [esphome-jk-bms](https://github.com/syssi/esphome-jk-bms) repository for more information.

## Requirements

* [ESPHome 2022.11.0 or higher](https://github.com/esphome/esphome/releases).
* Generic ESP32 DevKit-v1 30 pin board ( NOTE: ESP32-S2 currently has issues with CAN BUS and does not work! )
* For 5V CAN bus : TJA1050 or TJA1051T CAN controller interface module and 4.7K resistor for 5v to 3.3v level shifing.
* For 3.3V CAN bus : SN65HVD230 CAN controller interface module without 4.7K resistor
* Optional: 48V to 5V DC-DC converter to power the ESP32 from the JK-BMS VBAT pin (URB4805YMD-10WR3 or VRB4805S-6WR3)
* Optional: JK RS485 Adaptor and RS484 to TTL3.3v Adaptor (see optional schematic below)

## Schematics

![Image](images/PCB_ESP32_JK-BMS-CAN_powered_by_JK-BMS.png "PCB ESP32 JK-BMS-CAN powered by JK-BMS")

![Image](images/PCB_ESP32_JK-BMS-CAN_Prototype.jpg "PCB ESP32 JK-BMS-CAN powered by JK-BMS")

![Image](images/JK-BMS_24S_GPS_port.png "PCB ESP32 JK-BMS-CAN powered by JK-BMS")


```
3.3V CAN BUS

              RS485-TTL               RS232-TTL                CAN BUS (3V3)
┌──────────┐            ┌──────────┐             ┌────────────┐              ┌──────────┐
│          │<TX------RX>│16      23│<TX-------TX>|            |              |          |
│  JK-BMS  │<RX------TX>│17      22│<RX-------RX>| SN65HVD230 |<---CAN H --->| Inverter |
│          │<----GND--->│   ESP32  │<----GND---->|    CAN     |<---CAN L --->|          |
│          │     5V---->│VIN    3V3│<---3.3V---->|            |              |          |
└──────────┘            └──────────┘             └────────────┘              └──────────┘


5V CAN BUS

              RS485-TTL               RS232-TTL                CAN BUS (5V)
┌──────────┐            ┌──────────┐             ┌────────────┐              ┌──────────┐
│          │<TX------RX>│16      23│<TX-------TX>|            |              |          |
│  JK-BMS  │<RX------TX>│17      22│<RX--4K7--RX>|  TJA1050   |<---CAN H --->| Inverter |
│          │<----GND--->│   ESP32  │<----GND---->|    CAN     |<---CAN L --->|          |
│          │     5V---->│VIN    VIN│<----5V----->|            |              |          |
└──────────┘            └──────────┘             └────────────┘              └──────────┘


Optional below, as seen in pic above: RS485 between JK-BMS GPS port and ESP32, uses JK RS485 adaptor and RS485 to TTL3.3v adaptor.

              RS485-TTL                  RS485             RS485-TTL               RS232-TTL                CAN BUS
┌──────────┐            ┌───────────┐           ┌────────┐           ┌─────────┐             ┌─────────┐              ┌──────────┐
│          │<----TX---->│Y    JK   Y│<A------A+>│        │<TX-----RX>│16     23│<TX-------TX>|         |              |          |
│  JK-BMS  │<----RX---->│W  RS485  W│<B------B->│ RS485  │<RX-----TX>│17     22│<RX--4K7--RX>| TJA1050 |<---CAN H --->| Inverter |
│          │<----GND--->│B Adaptor B│<---GND--->│To 3.3V │<---GND--->|         |<----GND---->|   CAN   |<---CAN L --->|          |
│          │<----VBAT-->│R          │           │        │<---3.3V-->|  ESP32  |<----5V----->|         |              |          |
└──────────┘            └───────────┘           └────────┘           └─────────┘             └─────────┘              └──────────┘



# RS485-TTL jack on JK-BMS (4 Pin, JST 1.25mm pitch)
┌─── ─────── ────┐
│                │
│ O   O   O   O  │
│GND  RX  TX VBAT│ 
└────────────────┘
  │   │   │   | VBAT is full battery volatge eg 51.2V (No connect)
  │   │   └──── ESP32 GPIO16 (`rx_pin`)
  │   └──────── ESP32 GPIO17 (`tx_pin`)
  └──────────── GND
```


The UART-TTL (labeled as `RS485`) socket of the BMS can be attached to any UART pins of the ESP. A hardware UART should be preferred because of the high baudrate (115200 baud). The connector is called 4 Pin JST with 1.25mm pitch.

## Installation

This YAML file is based on the [@syssi](https://github.com/syssi) [esphome-jk-bms](https://github.com/syssi/esphome-jk-bms) repository.

```yaml
external_components:
  - source: github://syssi/esphome-jk-bms@main
```

**Installation procedure:**

```bash
# Install esphome
pip3 install esphome

# Clone this external component
git clone https://github.com/Sleeper85/esphome-jk-bms-can.git
cd esphome-jk-bms-can

# Create a secrets.yaml containing some setup specific secrets
cat > secrets.yaml <<EOF
wifi_ssid: MY_WIFI_SSID
wifi_password: MY_WIFI_PASSWORD

# Validate the configuration, create a binary, upload it, and start logs
esphome run esp32-jk-bms-can.yaml

# Optional add to Home Assistant
In Home Assistant under settings->Intergration "Add Intergration" select ESPHome add device jk-bms-can if found or supply ip address of ESP32

```

## ESPHome bash command

```bash
# test the config
esphome config esp32-jk-bms-can.yaml

# install the config in ESP32
esphome run esp32-jk-bms-can.yaml

# check the logs (the --device option is not required)
esphome logs esp32-jk-bms-can.yaml --device 192.168.x.x
```

## Known issues

* Fixed in version 1.15.1 - ~~ESP32 has a bug that causes WDT reboot if no other devices on CAN bus to ACK the packets. If you try to run without inverter it will not work as it will constantly WDT reboot!~~
* The battery type sensor is pretty useless because the BMS reports always the same value (`Ternary Lithium`). Regardless of which battery type was set / parameter set was loaded via the android app. ([#9][i9])
* ESP32: Adding all supported sensors can lead to a stack overflow / boot loop. This can be solved by increasing the stack size. ([#63][i63])

[i9]: https://github.com/syssi/esphome-jk-bms/issues/9
[i63]: https://github.com/syssi/esphome-jk-bms/issues/63

## Goodies

A user of the [@syssi](https://github.com/syssi) [esphome-jk-bms](https://github.com/syssi/esphome-jk-bms) project ([@dr3amr](https://github.com/dr3amr)) shared some [Home Assistant Lovelace UI cards for a beautiful dashboard here](https://github.com/syssi/esphome-jk-bms/discussions/230).

![Lovelace entities card](images/lovelace-cards-contribution.png "Home Assistant Lovelace UI cards")

## Debugging

If this component doesn't work out of the box for your device please update your configuration to enable the debug output of the UART component and increase the log level to the see outgoing and incoming serial traffic:

```
logger:
  level: DEBUG

uart:
  id: uart_0
  baud_rate: 115200
  rx_buffer_size: 384
  tx_pin: ${tx_pin}
  rx_pin: ${rx_pin}
  debug:
    direction: BOTH
```

## References

* https://www.patreon.com/Uksa007Codedevelopment Thanks to @uksa007 for making the original CAN code.
* https://github.com/syssi/esphome-jk-bms Thanks go to @syssi for help and making the original RS485 code.

