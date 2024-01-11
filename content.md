---
title: 'Configure a watchdog timer on the Finder Opta'
description: "Learn how to configure a watchdog timer on the Finder Opta to
              reboot it in case of malfunctions."
author: 'Fabrizio Trovato'
difficulty: intermediate
tags:
  - Watchdog
software:
  - ide-v1
  - ide-v2
  - arduino-cli
  - web-editor
hardware:
  - hardware/07.opta/opta-family/opta
---

## Overview

In this tutorial we are going to learn how to configure a watchdog timer on the
Finder Opta, in order to detect malfunctions such as infinite loops, deadlocks
and timeouts, and reboot the device.

## Goals

* Learn how to configure a watchdog timer on the Finder Opta.

## Required Hardware and Software

### Hardware Requirements

* Finder Opta PLC with RS-485 support (x1).
* USB-C® cable (x1).

### Software Requirements

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) or [Arduino Web
Editor](https://create.arduino.cc/editor).
* [Example code](assets/OptaWatchdog.zip).

## Finder Opta and watchdogs

Being an Mbed OS based board, the Finder Opta can leverage the [`Watchdog`
class](https://os.mbed.com/docs/mbed-os/v6.16/apis/watchdog.html) to set a
hardware watchdog timer that reboots the device in case of malfunctions.

## Instructions

### Setting Up the Arduino IDE

This tutorial will need [the latest version of the Arduino
IDE](https://www.arduino.cc/en/software). If it is your first time setting up
the Finder Opta, check out the [getting started
tutorial](/tutorials/opta/getting-started).

### Code Overview

The goal of the following tutorial is to configure a watchdog timer on the
Finder Opta, and to verify its functionality using some delays.

The full code of the example is available [here](assets/OptaWatchdog.zip):
after extracting the files the sketch can be compiled and uploaded to the
Finder Opta.

#### Configuring the watchdog

As explained during this tutorial, the Finder Opta can leverage the `Watchdog`
class provided by MbedOS to set a hardware timer that resets the device in case
of malfunctions.

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup()
{
    Serial.begin(9600);

    // Setup the watchdog to 10s.
    Serial.println("Setting watchdog to 10s.");
    mbed::Watchdog::get_instance().start(10000);
```

In the code above, the sketch gets the reference to the `Watchdog` instance of
MbedOS and starts it, with a maximum timeout of 10 seconds passed as parameter.

#### Refreshing thee watchdog

Next, the sketch will make the Finder Opta sleep for 5 seconds for three
consecutive times, refreshing the watchdog timer at the end of each:

```cpp
    // Sleep 5s and kick, for three times.
    for (int i = 0; i < 3; i++)
    {
        Serial.println("Sleeping for 5s.");
        delay(5000);
        mbed::Watchdog::get_instance().kick();
    }
```

Then the sketch will try to sleep for 11 seconds, triggering the watchdog and
causing the Finder Opta to reboot:

```cpp
    // Sleep 11s causing Opta to reboot.
    Serial.println("Sleeping for 11s. Opta will reboot.");
    delay(11000);
}
```

At this point we will see the LEDs on the Finder Opta blink as they usually do
after a manual reboot, and the sketch will start from the beginning. Note that,
during the execution of the program the Finder Opta will print on the serial
monitor the operation that it is executing, producing an output that should
look like the one below:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
```

## Conclusion

This tutorial shows how to configure a watchdog timer on the Finder Opta to
reboot the device in case of malfunctions, and then it verifies its
functionality using a sequence of delays.
