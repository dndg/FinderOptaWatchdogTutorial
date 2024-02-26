---
title: 'Configure a watchdog timer on the Finder Opta'
description: "Tutorial for configuring a watchdog timer on the Finder Opta to automatically reboot it in case of malfunctions."
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

The watchdog timer is a hardware component used to ensure the reliability of a
device by automatically restarting it in case of malfunctions or freezes. It is
a timer that must be regularly reset through a specific signal (in jargon, it
is said that this signal "feeds" the watchdog timer). If the timer reaches its
time limit before being "fed", it assumes there has been a system malfunction
and restarts the device to restore its proper operation.

In the case of the Finder Opta, the watchdog timer ensures continuous and
stable operation of the device. For instance, if we imagine using the Finder
Opta in a critical environment such as the control system of an industrial
plant, the watchdog timer will promptly intervene in case of software or
hardware malfunction, automatically minimizing downtime through device reboot.
This automated solution will allow bringing the system controlled by the Finder
Opta back to operational condition without physically accessing the device to
restart it, which is particularly useful in the embedded domain.

The use of a watchdog timer on the Finder Opta is considered a best practice
for developers, as it allows writing reliable and predictable programs,
simplifying maintenance.

## Goals

* Explain the importance and advantages of using a watchdog timer.
* Learn how to configure a watchdog timer on Finder Opta.

## Requirements

### Hardware

* Finder Opta PLC (x1).
* USB-C® cable (x1).

### Software

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) or [Arduino Web
Editor](https://create.arduino.cc/editor).

## Finder Opta and watchdogs

In the overview of this tutorial, we became acquainted with the concept of a
watchdog timer, explaining why it is a useful solution when programming a
Finder Opta.

As a device based on the Arduino Portenta H7 board, the Finder Opta can
leverage the
[`Watchdog`](https://os.mbed.com/docs/mbed-os/v6.16/apis/watchdog.html) class
from Mbed OS to set up a hardware watchdog timer. Furthermore, the same class
provides the `kick()` function, which takes care of "feeding" the watchdog.

## Instructions

### Setting Up the Arduino IDE

To follow this tutorial, you will need [the latest version of the Arduino
IDE](https://www.arduino.cc/en/software). If it's your first time setting up
the Finder Opta, we recommend taking a look at [Getting Started with
Opta](https://opta.findernet.com/it/tutorial/getting-started): in this
tutorial, we explain how to install the Board Manager for the `Mbed OS Opta`
platform, which is the set of basic tools necessary to create and use a sketch
for the Finder Opta with Arduino IDE.

### Code Overview

The purpose of the following tutorial is to configure a watchdog timer on the
Finder Opta, and then verify its functionality using programmed delays. In
particular:

* We will set up a watchdog timer for 10 seconds. This means that the Finder
  Opta will be restarted after 10 seconds of inactivity.
* We will test the watchdog by making the Finder Opta sleep for 5 seconds three
  consecutive times, feeding it at the end of each.
* Finally, we will make the Finder Opta sleep for 11 seconds. At this point,
  the watchdog timer will expire, causing the device to restart.

During this process, we will print some debug messages to the Arduino IDE
serial monitor, allowing us to understand in real-time what is happening on our
Finder Opta.

#### Configuring the watchdog

Let's start writing our sketch, beginning with the configuration of the
watchdog timer for 10 seconds.

The name of the sketch is `OptaWatchdog`, and the complete source code is
available at [this link](assets/OptaUpdater.zip). You can extract the content
of the `.zip` file and copy it into the `~/Documents/Arduino` folder.
Alternatively, you can create a new sketch named `OptaWatchdog` using the
Arduino IDE and paste the code provided in this tutorial.

Like all Arduino sketches, our program will consist of a `setup()` function and
a `loop()` function:

```cpp
void setup() {
  // Setup code, executed once.
}

void loop() {
  // Loop code, executed forever.
}
```

At the beginning of our sketch, we will import the necessary libraries:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  // Setup code, executed once.
}

void loop() {
  // Loop code, executed forever.
}
```

In particular, we have imported the following libraries:

* `Arduino`: contains numerous basic functionalities for Arduino boards, and
  it is therefore good practice to import it at the beginning of all sketches.
* `Watchdog`: MbedOS library used to access the watchdog timer of the Finder
  Opta, as mentioned earlier.

At this point, let's start writing our `setup()` code, which is executed once
by the Finder Opta. It should be noted that in this tutorial, it will suffice
to write only the code for `setup()`, leaving the `loop()` function empty. As
mentioned earlier we will let the watchdog timer of the Finder Opta expire,
causing it to restart, so the `loop()` code will never be executed.

The code below sets the baud rate of the serial communication to `9600` and
then configures the watchdog timer duration to 10 seconds:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Setup the watchdog to 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Code that sleeps for 5s and feeds the watchdog.

  // Code that sleeps for 11s causing Opta to reboot.
}

void loop() {
  // This code reaches the loop! If you use it and you do not
  // want Opta to reboot feed the watchdog here.
}
```

As indicated by the comment, if we were to execute the code above without first
writing the missing part of `setup()`, our program would reach the `loop()`
function. Since this function does not call `kick()`, the watchdog would
intervene after ten seconds from the last time it was fed, restarting Finder
Opta. This also helps us understand that if we decide to use a watchdog timer,
it will be our responsibility to regularly call the feed function to avoid
unwanted restarts.

Note that the duration is expressed in milliseconds, so we set a value of
`10000`. On the Arduino IDE serial monitor, we will see a message printed
indicating that the watchdog timer has been set. To allow time for the user to
start the Arduino IDE serial monitor and view this message, we introduce a 1
second delay at the beginning of `setup()`. Since the delay is introduced
before starting the watchdog timer, the operation does not affect the sketch in
any way.

#### Feeding the watchdog

Let's now add a loop that repeats three times, and within each iteration:

* Sleeps for 5 seconds.
* Feeds the watchdog using the `kick()` function provided by MbedOS.

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Setup the watchdog to 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Sleep 5s and feed the watchdog, for three times.
  for (int i = 0; i < 3; i++) {
    Serial.println("Sleeping for 5s.");
    delay(5000);
    mbed::Watchdog::get_instance().kick();
  }

  // Code that sleeps for 11s causing Opta to reboot.
}

void loop() {
  // This code reaches the loop! If you use it and you do not
  // want Opta to reboot feed the watchdog here.
}
```

During this phase, the watchdog timer will be fed regularly, preventing the
restart of the Finder Opta. We are simulating a situation of normal
operativity, where our device performs tasks and informs the watchdog that the
system is in a healthy state. On the Arduino IDE serial monitor, we will see an
output that looks like this:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
```

In this case as well, if we were to execute the code provided above without
completing the missing part of `setup()`, our program would reach the `loop()`
function, causing the Finder Opta to restart.

#### Watchdog intervention

At this point, we add the code that puts Finder Opta to sleep for 11 seconds,
simulating an operation that blocks the device for a longer time than the
maximum limit of 10 seconds (e.g., malfunction of the sketch, communication
timeout, infinite loop) set by the watchdog timer:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Setup the watchdog to 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Sleep 5s and feed the watchdog, for three times.
  for (int i = 0; i < 3; i++) {
    Serial.println("Sleeping for 5s.");
    delay(5000);
    mbed::Watchdog::get_instance().kick();
  }

  // Sleep 11s causing Opta to reboot.
  Serial.println("Sleeping for 11s. Opta will reboot.");
  delay(11000);
}

void loop() {
  // This code does not reach the loop.
}
```

On the Arduino IDE serial monitor, we will see an output that looks like this:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
```

Afterwards, the Finder Opta will reboot and our sketch will be executed from
the beginning. Using the Arduino IDE we can observe that for a brief moment our
Finder Opta will be automatically disconnected and then immediately
reconnected: this indicates the restart, easily inferred also from the output
on the serial monitor, which will repeat in a loop producing messages that look
like this:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
Setting watchdog to 10s.
Sleeping for 5s.
...
```

We have thus verified that our watchdog timer works correctly because it
restarts the board only when it is not fed for periods of time longer than 10
seconds. We reiterate once again that with our final `setup()` code, the
`loop()` function will never be reached and it is therefore unnecessary to
write its code.

Finally, we conclude by saying that the maximum limit value of the Finder Opta
watchdog is 32 seconds, and therefore it is not possible to use values greater
than this. We suggest not to set timers exceeding 30 seconds.

## Conclusion

In this tutorial, we discussed the importance of the watchdog timer on the
Finder Opta to ensure its reliability. We learned that the watchdog timer is a
critical component that constantly monitors the device and intervenes
automatically in case of malfunctions or freezes, restarting the Finder Opta.

Through a series of detailed instructions, we demonstrated how to configure a
watchdog timer on the Finder Opta using the Arduino IDE and the `Watchdog`
library from MbedOS. Subsequently, we saw how to periodically feed the watchdog
to prevent the automatic restart of the Finder Opta.

Finally, we tested the functionality of the watchdog timer by simulating
malfunction situations and verifying the proper reboot of the device. We
observed that the watchdog effectively intervenes only when the timer is not
reset within the predetermined time limit through the feed function.

In conclusion, the implementation of the watchdog timer on the Finder Opta is a
recommended best practice for developers as it helps ensure continuous and
reliable operation of the system in which the device is inserted, reducing
downtime automatically and without any manual intervention.
