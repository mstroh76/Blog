+++
author = "Martin Strohmayer"
title = "Mit PiBAN Festplatten löschen?"
date = "2021-06-30"
description = ""
featured = true
tags = [
    "Raspberry Pi"
]
categories = [
    "Raspberry Pi"
]
image = "images/PiBAN.jpg"
thumbnail = "images/PiBAN.jpg"
+++

Hin und wieder möchte man Festplatten sicher löschen. Mit dem Projekt iBAN und einer Raspberry Pi könnte es gelingen - oder etwa nicht?
<!--more-->

## Beschreibung

Öftmals liegt eine alte SATA-Festplatte herum und man würde sie ja gerne verkaufen oder verschenken. Aber es bleibt immer die Unsicherheit wegen der alten Daten. Kann man die vielleicht rekonstruieren? Dann vernichtet man die Platte doch lieber. Vielleicht wendet man sich sogar an eine offizielles Vernichtungsfirma. Dort kann man die Platte ja sogar mehrmals schreddern und den Rest dann gleich mitnehmen - Sicher ist Sicher (aber bitte bezahlen nicht vergessen). Nachzulesen auf https://de.wikipedia.org/wiki/Schredder-Aff%C3%A4re  .  
Wie gesagt, wenn man nicht ganz so hohe Standards hat, könnte ein sicheres Löschen auch ausreichen und die Hardware könnte weitverwendet werden. Wichtig zu sagen ist, dass sich das Verfahren nur auf HDD anwenden lässt, nicht auf die modernen SSD-Festplatten!

PiBAN ist eine Projekt das eine paar Scripts enthält. Wenn man diese Scripts auf einem Raspberry Pi mit Raspberry Pi OS installiert hat man eine automatische Löschstation.
Der Source liegt auf Github als Fork bei GC2 unter https://github.com/GrazerComputerClub/PiBAN .  

Ehrlich gesagt finde ich das Projekt zu gefährlich für eine dezidierte Station. Steckt man da mal eine falsche Festplatte an, weil man das Projekt vielleicht vergessen hat oder was auch immer, würde die Platte schon gelöscht werden. 
Ich habe deshalb davon abgesehen das Projekt so um zu setzen.  

Die Installation kann man im Script 'install.sh' sehen. Es werden die Tools mit ``apt install secure-delete nwipe`` installiert. Das Script 'update.sh' richtet die automatischen USB Funktionen ein, damit bei Anschluss alles sofort gestartet wird.  
Der eigentliche Schreddervorgang wird mit dem Script 'nuke.sh' gestartet.  Dort ist wiederum nur die Zeile ``shred -v --iterations=1 "$1"```  die wichtige Zeile.
Rund herum gibt es auch alternative Programmaufrufe zum sicheren Löschen mit nwipe. Wie dem Kommentar aber zu entnehmen ist sind die sehr bis 'unmöglich' langsam.  

Das Programm "shred" scheint hier eine Kompromiss zwischen Sicherheit und Geschwindigkeit zu sein. Näheres zum Programm findet man auf https://wiki.ubuntuusers.de/shred/ .



## Eigene Löschstation ohne PiBAN

Urspünglich wollte ich eine alte Raspberry Pi 1 für so eine Löschstation vorbereiten und zur Sicherheit den Vorgang nur über eine Taste und Display (4 Stellen Siebensegment) starten.

```
sudo apt -y install secure-delete wiringpi pv
```

/usr/local/bin/usbmount.sh:
```
#!/usr/bin/env bash
echo "Detected new device: $1" >>/var/log/PiBAN.log
devname=$(basename $1)
logname=/tmp/$devname.log
if [ "${ACTION}" = "add" ]
then
    echo $1 > /dev/shm/hdd
    2display Hdd
fi

if [ "${ACTION}" = "remove" ]
then
	/bin/rm /dev/shm/hdd
	2display noHd
fi
```


/usr/local/bin/shred.sh (gestartet aus rc.local oder systemd):
```
# 1: status led 
# 2: shred HDD if connected
gpio -g mode 2 in

echo shredstation started ... 
2display idle
while true; do
	GPIO=`gpio -g read 2`
	if [ $GPIO = 0 ]; then
		HDD=`cat /dev/shm/hdd` 
		echo found $HDD
		if [ ! -z "$HDD" ] ; then
			2display shrd
			shred -v -z -n 1 $HDD > /var/log/shred.log 2>&1
			2display done
		fi
	fi
	sleep 1
done
```

/etc/udev/rules.d/usbmount.rules:
```
ACTION=="add", KERNEL=="sd*[!0-9]", SUBSYSTEM=="block", RUN+="/usr/local/bin/usbmount.sh %N"
ACTION=="remove", KERNEL=="sd*[!0-9]", SUBSYSTEM=="block", RUN+="/usr/local/bin/usbmount.sh %N"
```

Scripts aktivieren:
```
udevadm control --reload-rules && udevadm trigger
sudo udevadm control --reload-rules
```

Leider ist es mir nicht gelungen den Prozentforschritt am Display anzuzeigen. Das Tool pv war nicht in der Lage den Output von shred korrekt auszuwerten ``sudo shred -n 1 -v /dev/sda | pv -n | 2display`` hat leider nicht funktioniert.


## Schreddern und Analyse

Prinzipiell eine gute Idee aber dann habe ich Performance Tests gemacht und analysiert wie lange so eine Löschung nun wirklich dauert. Denn eins habe ich nicht bedacht, die Löschung hängt von der USB Geschwindigkeit ab. Die ist bekanntlich bei allen Raspberry Pis bis 3+ auf USB 2.0 mit max. 30 MB/s begrenzt. Nimmt man nun den Raspberry Pi 4 mit der USB 3.0 Schnittstelle so erreicht er bis zu 100 MB/s also ca. 3-4 mal mehr.  
Als Festplatte verwendete ich eine 3 TB SATA Platte mit einer USB 3.0 Docking Station mit externer Stromversorgung.  


Schreddern, ein Durchgang:
```
sudo shred -n 1 -z -v /dev/sda 
```


Nur mit Nullen beschreiben:
```
shred -n 0 -z -v /dev/sda
```

Schreddern und dann mit Nullen beschreiben:
```
sudo shred -n 1 -z -v /dev/sda 
```

Das Schreddern dauerte mit dem Pi 1 (800 MHz) ca. 49 Stunden, also etwas mehr als 2 Tage. Mit dem Pi 4 dauerte es hingegen nur 11 Stunden, also rund einen halben Tag. Damit erreichte der Pi 1 ca. 17 MB/s und der Pi 4 ca. 78 MB/s.  
Das Nullsetzen der Platte dauerte mit dem Pi 1 ca. 44 Stunden und beim Pi 4 ca. 9 Stunden. Damit erreichte der Pi 1 ca. 19 MB/s und der Pi 4 ca. 95 MB/s.  
Zusammen würde als der Vorgang mit dem Pi 1 93 Stunden dauern, das sind dann fast 4 Tage!  
Beim Pi 4 wäre die 3TB Festplatte nach 20 Stunden gelöscht.
  
**Alles in allem macht eine Station für mich wenig Sinn, wenn nur einen 'edler' Pi 4 diese Aufgabe in akzeptabler Zeit erledigen kann. Damit macht das ganze PiBAN Projekt wenig Sinn und reduziert sich auf das Linux-Tool "shred".**  
Also stecke ich die Platte einfach an meinen Pi 4 an und starten die Löschung manuell. Für die wenige male die ich so eine Aufgabe benötige. Nachdem das alles schon so lange gedauert hat, habe ich mir den Test mit "nwipe" im übrigen erspart.
