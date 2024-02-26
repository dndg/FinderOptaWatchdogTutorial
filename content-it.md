---
title: 'Configurare un timer watchdog su Finder Opta'
description: "Tutorial per la configurazione di un timer watchdog su Finder Opta per riavvii automatici in caso di malfunzionamenti."
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

Il timer watchdog è un componente hardware utilizzato per garantire
l'affidabilità di un dispositivo e riavviarlo automaticamente in caso di
malfunzionamenti o blocchi. Si tratta di un timer che deve essere regolarmente
resettato tramite un apposito segnale di _feed_ (in gergo, si dice che questo
segnale "nutre" il timer watchdog). Se il timer raggiunge il proprio tempo
limite prima di essere "nutrito", il watchdog presume ci sia stato un
malfunzionamento del sistema e riavvia il dispositivo, per ripristinarne il
corretto funzionamento.

Nel caso di Finder Opta, il timer watchdog garantisce un'operatività continua e
stabile del dispositivo. Se immaginiamo infatti di utilizzare Finder Opta in un
ambiente critico come il sistema di controllo di un impianto industriale, il
timer watchdog interverrà tempestivamente in caso di malfunzionamento del
software o dell'hardware, minimizzando i tempi di blocco in maniera
completamente automatizzata tramite il riavvio del dispositivo. Questa
soluzione automatizzata permetterà di riportare il sistema pilotato da Finder
Opta in condizione di operatività senza dover accedere fisicamente al
dispositivo per riavviarlo, cosa particolarmente utile nell'ambito embedded.

L'utilizzo di un timer watchdog su Finder Opta è considerata una best practice
per gli sviluppatori, poichè permette di scrivere programmi affidabili e dal
comportamento prevedibile, semplificando la manutenzione.

## Obiettivi

* Spiegare l'importanza e i vantaggi dell'utilizzo di un time watchdog.
* Imparare a configurare un timer watchdog su Finder Opta.

## Requisiti

### Hardware

* PLC Finder Opta (x1).
* Cavo USB-C® (x1).

### Software

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) o [Arduino Web
Editor](https://create.arduino.cc/editor).

## Finder Opta e i watchdog

Nella panoramica di questo tutorial abbiamo familiarizzato con il concetto di
timer watchdog, spiegando perchè è una soluzione utile quando si programma un
Finder Opta.

Essendo un dispositivo basato sulla board Arduino Portenta H7, Finder Opta può
utilizzare la classe
[`Watchdog`](https://os.mbed.com/docs/mbed-os/v6.16/apis/watchdog.html) di Mbed
OS per impostare un timer watchdog in hardware. Inoltre, la stessa classe
fornisce la funzione `kick()` che si occupa di "nutrire" il watchdog.

## Istruzioni

### Configurazione dell'Arduino IDE

Per seguire questo tutorial, sarà necessaria [l'ultima versione dell'Arduino
IDE](https://www.arduino.cc/en/software). Se è la prima volta che configuri il
Finder Opta ti consigliamo di dare un'occhiata a [Getting Started with
Opta](https://opta.findernet.com/it/tutorial/getting-started): in questo
tutorial spieghiamo come installare il Board Manager per la piattaforma `Mbed
OS Opta`, ovvero l'insieme di tool di base necessari a creare e utilizzare uno
sketch per Finder Opta con Arduino IDE.

### Panoramica del codice

Lo scopo del seguente tutorial è di configurare un timer watchdog su Finder
Opta, andando poi a verificarne il funzionamento tramite dei delay programmati.
In particolare:

* Imposteremo un timer watchdog da 10 secondi. Ciò significa che Finder Opta
verrà riavviato in seguito a 10 secondi di inattività.
* Testeremo il watchdog facendo dormire Finder Opta per 5 secondi per 3 volte
consecutive, effettuando il _feed_ al termine di ciascuna.
* Faremo infine dormire Finder Opta per 11 secondi. A questo punto il timer
watchdog scadrà, causando il riavvio del dispositivo.

Durante questo processo, stamperemo sul monitor seriale di Arduino IDE alcuni
messaggi di debug che ci permettano di comprendere in tempo reale cosa sta
accadendo al nostro Finder Opta.

#### Configurare il watchdog

Iniziamo a scrivere il nostro sketch, partendo dalla configurazione del timer
watchdog da 10 secondi.

Il nome dello sketch è `OptaWatchdog`, il codice sorgente completo è
disponibile a [questo link](assets/OptaWatchdog.zip). È possibile estrarre il
contenuto del file `.zip` e copiarlo nella cartella `~/Documents/Arduino`, o
alternativamente creare un nuovo sketch chiamato `OptaWatchdog` utilizzando
Arduino IDE ed incollare il codice presente in questo tutorial.

Come tutti gli sketch per Arduino, il nostro programma sarà composto da una
funzione `setup()` e una funzione `loop()`:

```cpp
void setup() {
  // Codice di setup, eseguito all'avvio.
}

void loop() {
  // Codice di loop, eseguito all'infinito.
}
```

All'inizio del nostro sketch andremo ad importare le librerie necessarie al
funzionamento del programma:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  // Codice di setup, eseguito all'avvio.
}

void loop() {
  // Codice di loop, eseguito all'infinito.
}
```

In particolare abbiamo importato le librerie:

* `Arduino`: contiene numerose funzionalità di base per le schede Arduino, ed è
  quindi buona norma importarla all'inizio di tutti gli sketch.
* `Watchdog`: libreria di MbedOS utilizzata per accedere al tiemr watchdog di
  Finder Opta, come detto in precedenza.

A questo punto iniziamo a scrivere il nostro codice di `setup()`, eseguito una
singola volta da Finder Opta. Si noti che in questo tutorial sarà sufficiente
scrivere solamente il codice di `setup()`, lasciando la funzione `loop()`
vuota. Infatti, come detto in precedenza faremo scadere il timer watchdog di
Finder Opta causandone il riavvio. Pertanto, una volta terminato lo sketch, il
codice di `loop()` non verrà mai eseguito.

Il codice qui sotto imposta la velocità di trasmissione della comunicazione
seriale a `9600` e in seguito imposta la durata del timer watchdog a 10
secondi:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Imposta timer watchdog a 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Codice che dorme per 5 secondi e nutre il watchdog.

  // Codice che dorme per 11 secondi e fa riavviare Opta.
}

void loop() {
  // Questo codice raggiunge il loop! Se lo utilizzi e non vuoi
  // riavviare Opta nutri il watchdog in questo punto.
}
```

Come indicato dal commento, se dovessimo eseguire il codice inserito qui sopra
senza prima scrivere la parte di `setup()` mancante, il nostro programma
raggiungerebbe la funzione `loop()`. Poichè questa funzione non chiama
`kick()`, il watchdog interverrebbe dopo dieci secondi dall'ultima volta che è
stato nutrito, riavviando Finder Opta. Questo ci permette anche di capire che
se decidiamo di utilizzare un timer watchdog all'interno di uno sketch, sarà
nostra premura chiamare la funzione di _feed_ con regolarità per evitare
riavvii indesiderati.

Si noti che la durata del timer watchdog è espressa in millisecondi, pertanto
impostiamo un valore di `10000`. Sul monitor seriale di Arduino IDE vedremo
stampato un messaggio che indica che è stato impostato il timer watchdog; per
permettere di visualizzare questo messaggio, ad inizio `setup()` introduciamo
un delay di 1 secondo che dia all'utente il tempo necessario ad avviare il
monitor seriale di Arduino IDE. Siccome il delay viene introdotto prima di
avviare il timer watchdog, il funzionamento dello sketch non è influenzato in
alcun modo.

#### Nutrire il watchdog

Aggiungiamo ora un loop che si ripete 3 volte e ciascuna volta:

* Dorme per 5 secondi.
* Nutre il watchdog con la funzione di feed `kick()`, fornita da MbedOS.

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Imposta timer watchdog a 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Dormi 5 secondi e nutri il watchdog, per tre volte.
  for (int i = 0; i < 3; i++) {
    Serial.println("Sleeping for 5s.");
    delay(5000);
    mbed::Watchdog::get_instance().kick();
  }

  // Codice che dorme per 11 secondi e fa riavviare Opta.
}

void loop() {
  // Questo codice raggiunge il loop! Se lo utilizzi e non vuoi
  // riavviare Opta nutri il watchdog in questo punto.
}
```

Durante questa fase, il timer watchdog sarà nutrito con frequenza, evitando il
riavvio di Finder Opta. Stiamo quindi simulando una situazione di normale
operatività, in cui il nostro dispositivo esegue dei task e segnala al watchdog
che il sistema è in stato di salute. Sul monitor seriale di Arduino IDE vedremo
un output di questo tipo:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
```

Anche in questo caso se dovessimo eseguire il codice inserito qui sopra senza
terminare la parte di `setup()` mancante, il nostro programma raggiungerebbe la
funzione `loop()` causando il riavvio di Finder Opta.

#### L'intervento del watchdog

A questo punto aggiungiamo il codice che fa dormire Finder Opta per 11 secondi,
simulando un'operazione che blocca il dispositivo per un tempo maggiore del
limite massimo di 10 secondi (es. malfunzionamento dello sketch, timeout di
comunicazione, loop infinito) a cui abbiamo impostato il timer del watchdog:

```cpp
#include <Arduino.h>
#include <drivers/Watchdog.h>

void setup() {
  Serial.begin(9600);
  delay(1000);

  // Imposta timer watchdog a 10s.
  Serial.println("Setting watchdog to 10s.");
  mbed::Watchdog::get_instance().start(10000);

  // Dormi 5 secondi e nutri il watchdog, per tre volte.
  for (int i = 0; i < 3; i++) {
    Serial.println("Sleeping for 5s.");
    delay(5000);
    mbed::Watchdog::get_instance().kick();
  }

  // Dormi per 11 secondi e fai riavviare Opta.
  Serial.println("Sleeping for 11s. Opta will reboot.");
  delay(11000);
}

void loop() {
  // Questo codice non raggiunge il loop.
}
```

Sul monitor seriale di Arduino IDE vedremo un output di questo tipo:

```text
Setting watchdog to 10s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 5s.
Sleeping for 11s. Opta will reboot.
```

In seguito Finder Opta si riavvierà e il nostro sketch verrà eseguito da capo.
Tramite Arduino IDE possiamo notare che per qualche istante il nostro Finder
Opta verrà automaticamente disconnesso ed in seguito immediatamente riconnesso:
questo ne segnala il riavvio, facilmente intuibile anche dall'output presente
sul montior seriale, che si ripeterà in loop ad ogni riavvio assumendo un
aspetto simile a questo:

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

Abbiamo quindi verificato che il nostro timer watchdog funziona correttamente,
poichè riavvia la scheda solamente quando non viene nutrito per periodi di
tempi superiori a 10 secondi. Ribadiamo ancora una volta che con il nostro
codice di `setup()` terminato, la funzione di `loop()` non verrà mai raggiunta
ed è pertanto inutile scriverne il codice.

Concludiamo infine dicendo che il valore limite del watchdog di Finder Opta è
di 32 secondi, e non è quindi possibile utilizzare valori maggiori. Suggeriamo
pertanto di non impostare timer superiori ai 30 secondi.

## Conclusioni

In questo tutorial abbiamo discusso dell'importanza del timer watchdog su
Finder Opta per garantirne l'affidabilità. Abbiamo imparato che il timer
watchdog è un componente critico che monitora costantemente il dispositivo e
interviene automaticamente in caso di malfunzionamenti o blocchi, riavviando
Finder Opta. Attraverso una serie di istruzioni dettagliate, abbiamo mostrato
come configurare un timer watchdog su Finder Opta utilizzando l'Arduino IDE e
la libreria `Watchdog` di MbedOS. In seguito, abbiamo visto come "nutrire"
periodicamente il watchdog per evitare il riavvio automatico di Finder Opta.
Infine, abbiamo testato il funzionamento del timer watchdog simulando
situazioni di malfunzionamento e verificando il corretto riavvio del
dispositivo. Abbiamo osservato che il watchdog interviene efficacemente solo
quando il timer non viene resettato entro il tempo limite prestabilito tramite
la funzione di _feed_.

In conclusione, l'implementazione del timer watchdog su Finder Opta è una best
practice consigliata per gli sviluppatori in quanto contribuisce a garantire
un'operatività continua e affidabile del sistema in cui viene inserito il
dispositivo, riducendo i tempi di inattività in maniera automatica e senza
alcun intervento manuale.
