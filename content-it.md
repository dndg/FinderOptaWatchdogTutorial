---
title: 'Configurare un timer watchdog su Finder Opta'
description: "Imparare a configurare un timer watchdog su Finder Opta per
              riavviarlo in caso di malfunzionamenti."
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

## Panoramica

In questo tutorial impareremo a configurare un timer watchdog su Finder Opta,
in modo da riavviare il dispositivo in caso di malfunzionamenti come loop
infiniti, deadlock o timeout.

## Obiettivi

* Imparare a configurare un timer watchdog su Finder Opta.

## Requisiti hardware e software

### Requisiti hardware

* PLC Finder Opta (x1).
* Cavo USB-C® (x1).

### Requisiti Software

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) o [Arduino Web
Editor](https://create.arduino.cc/editor).
* [Codice di esempio](assets/OptaWatchdog.zip).

## Finder Opta e i watchdog

Essendo una board basata su Mbed OS, Finder Opta può sfruttare la classe
[`Watchdog`](https://os.mbed.com/docs/mbed-os/v6.16/apis/watchdog.html) per
impostare un timer watchdog in hardware che riavvii il sistema in caso di
malfunzionamenti.

## Istruzioni

### Configurazione dell'Arduino IDE

Per seguire questo tutorial, sarà necessaria [l'ultima versione dell'Arduino
IDE](https://www.arduino.cc/en/software). Se è la prima volta che configuri il
Finder Opta, dai un'occhiata al tutorial [Getting Started with
Opta](/tutorials/opta/getting-started).

### Panoramica del codice

Lo scopo del seguente tutorial è di configurare un timer watchdog su Finder
Opta, andando poi a verificarne il funzionamento tramite dei delay programmati.

Il codice completo di esempio è disponibile [qui](assets/OptaWatchdog.zip).
Dopo aver estratto i file, lo sketch può essere compilato e caricato sul Finder
Opta.

#### Configurare il watchdog

Come specificato all'inizio di questo tutorial, Finder Opta può utilizzare la
classe `Watchdog` di MbedOS per impostare un timer hardware che riavvii il
dispositivo in caso di malfunzionamenti.

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

Nel codice riportato qui sopra, lo sketch ottiene una referenza al `Watchdog`
di MbedOS e lo avvia, con tempo di timeout massimo di 10 secondi.

#### Rinfrescare il watchdog

In seguito lo sketch fa dormire Finder Opta per 5 secondi per tre volte
consecutive, andando a rinfrescare il timer watchdog al termine di ciascuna:

```cpp
    // Sleep 5s and kick, for three times.
    for (int i = 0; i < 3; i++)
    {
        Serial.println("Sleeping for 5s.");
        delay(5000);
        mbed::Watchdog::get_instance().kick();
    }
```

Infine lo sketch prova a dormire per 11 secondi, attivando così il watchdog e
causando il riavvio di Finder Opta:

```cpp
    // Sleep 11s causing Opta to reboot.
    Serial.println("Sleeping for 11s. Opta will reboot.");
    delay(11000);
}
```

A questo punto vedremo i LED di Finder Opta lampeggiare come in seguito ad un
riavvio manuale, e lo sketch verrà eseguito da capo. Si noti che, durante
l'esecuzione del programma Finder Opta stamperà su monitor seriale l'operazione
che sta eseguendo, producendo un output di questo tipo:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
```

## Conclusioni

Questo tutorial mostra come configurare un timer watchdog su Finder Opta per
riavviare il dispositivo in caso di malfunzionamenti, andando poi a verificarne
il funzionamento tramite una sequenza di delay.
