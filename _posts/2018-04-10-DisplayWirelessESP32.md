---
layout: page
#
# Content
#
subheadline: "IoT"
title: "Un display OLED WiFi con ESP32"
teaser: "Come utilizzare un modulo ESP32 + display OLED per visualizzare dati via WiFi"
meta_description: "Come utilizzare un modulo ESP32 + display OLED per visualizzare dati via WiFi"
categories: 
  - hacking
tags:
  - hacking
  - electronics
  - ESP32
  - MQTT
  - IoT
  - MicroPython
#
# Styling
#
header:
  image_fullwidth: "/2018-04-10-DisplayWirelessESP32/board.jpg"

image:
  title: "/2018-04-10-DisplayWirelessESP32/board.jpg"
  thumb: "/2018-04-10-DisplayWirelessESP32/board-thumb.jpg"
  caption: ""
  caption_url: ""


---

Per monitorare il funzionamento del mio impianto fotovoltaico, ho installato da qualche anno un sistema chiamato
[OpenEnergyMonitor](http://openenergymonitor.org), basato su [Raspberry Pi](http://www.raspberrypi.org) e [Arduino](http://www.arduino.cc).

Il software di gestione di OpenEnergyMonitor utilizza un broker [MQTT](http://mqtt.org), 
chiamato [Mosquitto](https://mosquitto.org/), che permette ad altri software di connettersi e ricevere e trasmettere messaggi in modalità <i>publish/subscribe</i> in
modo efficiente e con ridotto consumo di banda e di energia.

I valori letti vengono pubblicati periodicamente, e possono essere visualizzati per esempio utilizzando l'utility <code>mosquitto_sub</code>
(le credenziali di default sono username <code>emonpi</code> e password <code>emonpimqtt2016</code>), registrandosi come <i>subscriber</i> per
il topic generico, indicato in MQTT con <code>#</code>:
<pre>
$ mosquitto_sub -h emonpi -u emonpi -P emonpimqtt2016 -v -t '#'
/caldaia/barometer/t 19.14
/caldaia/barometer/p 1005.46
emon/emonpi/power1 260
emon/emonpi/power2 17
emon/emonpi/power1pluspower2 277
emon/emonpi/vrms 247.25
emon/emonpi/t1 19
emon/emonpi/t2 0
emon/emonpi/t3 0
emon/emonpi/t4 0
emon/emonpi/t5 0
emon/emonpi/t6 0
emon/emonpi/pulsecount 0
</pre>

In questo caso le opzioni sono:
<ul>
<li><code>-h emonpi</code> per connettersi all'host <code>emonpi</code>, che è il nome DNS di default che il sistema OpenEnergyMonitor si assegna</li>
<li><code>-u emonpi</code> per il nome utente</li>
<li><code>-P emonpimqtt2016</code> per la password</li>
<li><code>-v</code> per far stampare accanto ad ogni dato il <i>topic</i> MQTT</li>
<li><code>-t '#'</code> per iscriversi a tutti i topic pubblicati dal broker</li>
</ul>

e l'output consiste in due colonne:
<ul>
<li>nella prima colonna, il <i>topic</i> MQTT</li>
<li>nella seconda colonna, il valore ricevuto</li>
</ul>
in questo caso, si può vedere come vengano ricevuti valori pubblicati direttamente da emonpi (es. la potenza istantanea prodotta e consumata), sia quelli che vengono inviati da un'altro dispositivo
che ho installato presso la caldaia per misurare la temperatura esterna e la pressione atmosferica.

Per visualizzare in tempo reale i valori del sistema, è possibile utilizzare una scheda con [ESP32](https://www.espressif.com/en/products/hardware/esp32/overview), un
SOM che supporta WiFi e Bluetooth Low-Energy, alla quale è collegato un display OLED. Una di queste schede è la [D-duino-32](https://www.tindie.com/products/lspoplove/wifi-packet-monitorv3-preflashed-d-duino-32-sdv2/),
che ha installato sulla stessa basetta un display OLED SSD1306 da 128x64 pixel, può essere alimentato con un qualsiasi alimentatore a 5V con connessione micro-USB (es. quelli dei telefoni) e ha dimensioni ridottissime: circa 6x3cm.

Per programmare la scheda, ho utilizzato [MicroPython](https://micropython.org/), nella sua versione per ESP32, che allo stato attuale è disponibile solo come [nightly build](http://micropython.org/download#esp32).

Il firmware può essere installato utilizzando il software <code>esptool.py</code> (che si può ottenere con <code>pip</code>: <code>pip install esptool</code>), con il comando
<pre>
$ esptool.py --port /dev/ttyUSB0 erase_flash
$ esptool.py --port /dev/ttyUSB0 write_flash -z 0x1000 firmware.bin
</pre>

A questo punto, è possibile utilizzare l'interprete Python installato sulla scheda direttamente con un emulatore di terminale seriale, come [Putty](https://www.putty.org/) o minicom.
Per conoscere le librerie specifiche per ESP32 si può consultare la [documentazione di Micropython](https://docs.micropython.org/en/latest/esp8266/)

MicroPython all'avvio esegue, se presenti, prima il file <code>boot.py</code>, quindi il file <code>main.py</code>.

Il file possono essere caricati sulla scheda utilizzando l'utility di Adafruit <a href="https://github.com/adafruit/ampy"><code>ampy</code></a>, che può anch'esso essere installata utilizzando <code>pip</code> con il comando
<code>pip install adafruit-ampy</code>:
<pre>
$ ampy --port /dev/ttyUSB0 put main.py
</pre>

Occorrono inoltre la [libreria Adafruit per il display SSD1306](https://github.com/adafruit/micropython-adafruit-ssd1306), che può essere installata utilizzando <code>ampy</code>, e le
librerie per MQTT che si possono installare direttamente da terminale sulla scheda utilizzando <code>upip</code>:
<pre>
import upip

upip.install('micropython-umqtt.simple')
upip.install('micropython-umqtt.robust')
upip.install('micropython-umqtt.simple') # Seconda chiamata identica per un bug di upip
</pre>

I codici sorgenti utilizzati per visualizzare i dati si possono recuperare nel [progetto GitHub](https://github.com/mfortini/ESP32Display).

Quando il sistema viene alimentato:
<ol>
<li>Per prima cosa, cerca la connessione alla rete</li>
<li>Quando la connessione è stabilita, recupera l'orario corretto via NTP</li>
<li>Si connette al server MQTT, chiedendo di iscriversi ad alcuni dei valori pubblicati (temperature, pressione, potenza istantanea)</li>
<li>Entra in un loop nel quale controlla ogni 0.5 secondi se ci sono messaggi MQTT, nel qual caso aggiorna i valori da visualizzare sul display</li>
<li>I valori visualizzati sul display vengono aggiornati</li>
<li>Se la connessione wifi cade, il sistema ritenta la connessione</li>
</ol>

Purtroppo micropython utilizza un font per il display che non è particolarmente leggibile.

Abbiamo quindi ottenuto un display che, ovunque alimentato in copertura della rete wifi, può visualizzare i valori istantanei del nostro sistema. Il risultato è quello che si può vedere in figura.
