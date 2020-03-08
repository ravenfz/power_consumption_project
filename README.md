# Power Consumption Meter Project

## 1 - Introduction

The purpose of this repo is to describe the steps of creating a budget power consumption meter for your home (or any circuit up to 100A).

This is part of a bigger project (under development) for the installation of a self-production energy module. Measure power consumption is part of the initial assessment for proper dimensioning of the system and, moving forward, to measure the production as well.

Note: All work was done using MacOS and it will be the reference in this project.

## 2 - Architecture

### 2.1 - Overview

There are two main functionalities required for the Power Consumption meter: measure and display information.
For the measure part, after some search, the module Peacefair PZEM-004T kept poping up. Its a module capable of readings up to 100A using either a whole or split core.

For more versatability and mobility with the PZEM-004T, most solutions seen would join it with anything with an ESP8266, such as a Sonoff Basic, for easy programability and WiFi capabilities.
The Sonoff Basic was chosen as it can easely be flashed with a custom firmware (Tasmota) which would bring the programability and functionalities lacking in the PZEM-004T and also proven to be a cost efective hardware with an ESP8266.

The meaurement part is therefore accomplished by using the PZEM and the Sonoff integrated, providing not only the measurement part but also integrated with an MQTT Client (within Tasmota) capable to send data to a MQTT Broker.

For data storage and visualization the option fell within Hass.io HomeAssistant installed on a (reporpused) Raspberry Pi 3.
HomeAssistant seems to have a growing community with plenty of plugins for a multitude of porpuses, mainly focused on Home Automation but also capable of data storage and viewability.

### 2.2 - BoM (Bill of Materials)

This is a full bill of materials used but only a few were bought for the project as, mostly on the RPI side, I had everything (all items unitary unless otherwise mentioned).

| Item  |  Short Description  |   Bought For Project?  |
| ------------------- | ------------------- | ------------------- |
|  Raspberry Pi 3+  |  Module to run HomeAssistant | No |
|  RPI Power Module 2.4V  |  Power the RPI | No |
|  16GB Micro SD Card  |  Required for OS install/Run | No |
|  PZEM-004T with Split-Core  |  Power Measurement | Yes |
|  Sonoff Basic  |  WiFi Module | Yes |
|  1k Ohm Resistance  |  Required PZEM004 and Sonoff integration | Yes |
|  PCB Pin Headers (x 4)  |  To Solder to Sonoff Basic's GPIOs | Yes |
|  Dupont Cable F2F (x 4)  |  To Solder to Sonoff Basic's GPIOs | No |
|  USB To TTL Adapter  |  Used to flash Sonoff Basic | Yes |
|  Schuko Plug with 2m wire  |  Power Supply for PZEM/Sonoff | Yes |

NOTE: Not considering any tools used (multimeter, wire crimpers/clipers, solder iron etc)

Overall spent around 16€ with all the materials bought for the project. Of course the most expensive component is the RPI (and related items) which I already had but can add an extra 50€.

### 2.3 - Sonoff Basic

Sonoff Basic is a well known module used to create a switch within any circuit (<10A) for automation. Its a rather cheap module that brings with it a ESP8266 module which means it is flashable with [Tasmota](https://github.com/arendst/Tasmota), a custom firmware for ESP8266 modules.

The Sonoff Basic has currently 3 known hardware revisions: R1, R2 and R3.

**The HW version used in this project was R2.** The versions can be easely recognized following the instructions on [this link](https://github.com/arendst/Tasmota/wiki/Sonoff-Basic).

**Why Tasmota?**

We will need both to communicate with the PZEM-004T module through the TTL Serial interface and on the other end to deliver the data to HomeAssistant MQTT Broker and the original firmware is not suited for that purpose.

On top of that, Tasmota has all the features the original firmware has (i.e.: switch) when accessed directly or with HomeAssistant.

### 2.4 - PZEM-004T

The Peacefair PZEM-004T is a reader of electrical related indicators such as Voltage, Current and Frequency. Its capable of up to 10 Amps without any external measurement aid and if used a closed or split coil it can go up to 100 Amps.

For this project it was used a PZEM-004T version 3 (as oposed to older Version 1) and a Split Coil transformer.

There is a great website with tons of information regarding the PZEM, including the data-sheet and wiring information available at [InnovatorsGuru](https://www.innovatorsguru.com/pzem-004t-v3/).

### 2.5 - Home Assistant

Home Assistant calls itself the "ultimate home automation hub". It is an open source software with a active and growing community that features an easy to install automation platform for devices that somehow provide an API that can be used. It is also usefull as a data broker (MQTT Broker) and storage/visualization either out of the box or with its plugins (such as InfluxDB and Grafana).

For this project we will use just set it up and configure as MQTT Broker for the data coming from the PZEM/Tasmota and provide some level of data visualization.

## 3 - Implementation

This section will detail de steps (or where to find them) for setup each component of the solution and in the end cover the integration of all the parts.

### 3.1 - Sonoff Basic

#### 3.1.1 - Soldering Pin Heads to the PCB GPIO's

Some of the GPIOs in this board will actively be used both to flash and to connect to PZEM Module for data communication.

Its adviced to solder 4 pin heads to the required ports before moving forward.

If you turn the Sonoff upside down you will see in the midle of the PCB 4 GPIOs named: GND, TX, RX and 3V3. These are the ones to have soldered pin heads.

![Sonoff_Basic_upside_down_PCB](https://raw.githubusercontent.com/reloxx13/reloxx13.github.io/master/media/tasmota/sonoff-basic-r2/back.jpg "Sonoff_Basic_upside_down_PCB")
*Sonoff Basic R2 rear PCB view and location of the 4 GPIOs. Image from [Tasmota Github](https://github.com/arendst/Tasmota/wiki/Sonoff-Basic)*

After soldering it should look something like this.

**TODO
[INSERT IMAGE]**

#### 3.1.2 - Backup Current Sonoff Firmware (optional)

This step is optional not only because you might not want to get back to the previous original firmware but also because there is a page which provides a guide and some backups from Sonoff devices, including the Sonoff Basic R2.
That page is [Hobbytronics Blog](https://blog.hobbytronics.pk/sonoff-original-firmware-backup-restore/#Backup-Download-Official-Firmware).

**Pre-Requirements**

Install Python on your laptop.

1. Install esptool:

    ````
    $ pip install esptool
    ````

2. Set the ttl-USB dongle to 3V (usually has a jumper to select between 5V and 3V);

3. Wire the TTL-USB dongle to sonoff:

   | USB TTL |	Sonoff |
   | --| -- |
   GND |	GND
   VCC	| 3V3
   TXD	| RX
   RXD |	TX

4. Connect the TTL-USB and Sonoff in programming mode:

   Press the black button on Sonoff and connect the TTL-USB device to the MAC, after connecting, release the black button

5. Find which USB port is the TTL-USB device (should see 2 with same ID, refers to same device)

   ````
   $ ls /dev/ | grep serial
   cu.usbserial-14110
   tty.usbserial-14110
   `````

6. Execute the esptool comand to backup the firmware:

    ````
    $ esptool.py --port /dev/tty.usbserial-14110 read_flash 0x00000 0x100000 sonoff_basic_R2image1M.bin


    esptool.py v2.8
    Serial port /dev/tty.usbserial-14110
    Connecting....
    Detecting chip type... ESP8266
    Chip is ESP8285
    Features: WiFi, Embedded Flash
    Crystal is 26MHz
    MAC: 24:62:ab:4b:ac:11
    Uploading stub...
    Running stub...
    Stub running...
    1048576 (100 %)
    1048576 (100 %)
    Read 1048576 bytes at 0x0 in 95.7 seconds (87.6 kbit/s)...
    Hard resetting via RTS pin...
    ````
7. The operation should be complete within couple of minutes. Validate that the file sonoff_basic_R2image1M.bin is in your filesystem.

#### 3.1.3 - Flash Tasmota Firmware

1. Here all I have to say is to follow the instructions from [this excelent Medium Post from Jordan Rounds](https://medium.com/@jordanrounds/sonoff-basic-r2-tasmota-aa6f9d4e033f) untill the section "Open Tasmota".
 

#### 3.1.4 - Configure Tasmota

You should have followed all the steps in the previous section untill the point your Sonoff/Tasmota is connected to your wireless network and has a certain known IP Address (it is up tou you to provide a static IP on your router if you desire so).

1. Access to your Tasmota through a webrowser using it's IP Address;


### 3.2 - PZEM-004

### 3.3 - Home Assistant

### 3.4 - Integration





##  - Ackowledgments