# Power Consumption Meter Project

## 1 - Introduction

The purpose of this repo is to describe the steps of creating a budget power consumption meter for your home (or any circuit up to 100A).

This is part of a bigger project (under development) for the installation of a self-production energy module. Measure power consumption is part of the initial assessment for proper dimensioning of the system and, moving forward, to measure the production as well.

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
|  PCB Pin Headers (x 4)  |  Soldered to Sonoff Basic | Yes |
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



## 3 - Implementation

### 3.1 - Sonoff Basic

### 3.2 - PZEM-004

### 3.3 - Home Assistant

### 3.4 - Integration





##  - Ackowledgments