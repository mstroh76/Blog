+++
author = "Martin Strohmayer"
title = "Raspberry Pis verwalten mit Ansible"
date = "2021-06-02"
description = ""
featured = true
tags = [
    "Raspberry Pi"
]
categories = [
    "Raspberry Pi"
]
image = "images/Ansible.jpg"
thumbnail = "images/Ansible.jpg"
+++

Wer in seinem Netwerk mehrere Raspberry Pi laufen hat und diese gemeinsam verwalten will, kann mit Ansible diese Aufgabe erledigen ...
<!--more-->


## Beschreibung

Inzwischen haben immer mehr Leute mehrere Raspberry Pis in ihrem Netzwerk laufen. Eine für die Homeautomation, eine als NAS-Server und vielleicht noch eine mit Kameraaufgaben wie Überwachung oder für die Vögelbeobachtung. Hin und wieder möchte man die Rapsberry Pis dann aber auch aktualisieren oder andere Aktien auf allen durchführen. Einzel kann das dann zur mühevollen Aufgabe werden. Ansible kann hier allerdings einiges vereinfachen und einzelne Befehle oder Befehlsequenzen auf allen System ausführen.


## Vorbereitung: SSH Zugriff ohne Passwort

Damit man von einem zentralen PC oder Laptop ohne Passworteingabe auf den Raspberry Pi zugreifen kann muss SSH korrekt eingerichtet sein. Der SSH-Key vom Host muss auf allen Clients (Raspberry Pis) übertragen werden. Dazu muss am Host der SSH-Key einmalig erstellt werden. Das macht man mit dem Befehl ssh-keygen. Man kann hier dann eine Passwort vergeben oder auch leer lassen - wie man möchte. Danach kann der SSH-Key auf die Clients übertragen wwerden. Das macht man am besten mit ``ssh-copy-id`` gefolgt vom username pi und hostname des Clients.

``` 
ls ~/.ssh/id_*.pub || ssh-keygen
ssh-copy-id pi@homeautomation
ssh-copy-id pi@piserver
ssh-copy-id pi@picamera01
``` 

## Ansible Installation



Nun kann Ansible mit ``apt install ansible`` installiert werden. Dies ist allerdings die etwas ältere Version 2.7.7. Die aktuellste könnte mit pip3 installiert werden. Ich habe allerdings darauf verzichtet und benutze die Version Der Distribution. 

``` 
sudo apt-get install python3-pip
pip3 install --user ansible
``` 

## Ansible Konfiguration

Nun gibt man die Host in der Datei "etc/ansible/hosts" an. Man kann sie gleich in Gruppen zusammenfassen um später Aktionen für alle Clients einer Gruppe zu starten. 
 
``` 
[pis]
homeautomation
piserver
picamera01

[picams]
picamera01
```
    

Mit dem Befehl ``ansible all --list-hosts`` können alle Hosts aufgelistet und damit die Konfigurationsdatei überprüft werden.

```
  hosts (3):
    homeautomation
    piserver
    picamera01
```


Da wir nur mit Raspberry Pis arbeiten, kann man als user pi verwenden. Das hat Gegenüber root den Nachteil, dass man Befehle oft mit sudo ausführen muss.
Allerdings entspricht der pi Benutzer dem üblichen Vorgang. 
Der Benutzer für den SSH Zugang wird in die Datei "/etc/ansible/ansible.cf" eingetragen

``` 
 remote_user = pi
``` 

Nun kann man bereits den ersten Befehl ``ansible all -m ping`` ausprobieren. Wenn alles korrekt eingerichtet und die SSH-Keys übertagen wurden wird der Befehl erfolgreich ausgeführt.


```
piserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
homeautomation | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
picamera01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Ansible Playbook

Playbooks sind Anweisungs-Dateien die ein oder mehrere Befehlr enthalten, die auf den Clients ausgeführt werden sollen. Als Beispiel gibt es ein upgrade Playbook und ein reboot Playbook.  

**Playbook upgrade:**
``` 
---
- hosts: all
  tasks:
    - name: Wait for automatic system updates
      shell: while sudo fuser /var/lib/dpkg/lock >/dev/null 2>&1; do sleep 1; done;
    - name: apt-get update && apt-get upgrade
      sudo: yes
      apt:
        upgrade: "yes"
        update_cache: "yes"
        cache_valid_time: 0
``` 

**Playbook reboot:**
``` 
---
- hosts: all
  tasks:
    - name: Reboot host
      sudo: yes
      shell: sleep 2 && /sbin/shutdown -r now "Ansible reboot"
      async: 1
      poll: 0
    - name: Wait for host to come back up
      become: false
      wait_for_connection:
        delay: 15
        sleep: 2
        timeout: 120
``` 

Nun kann man die Playbooks ausführen: 

```
ansible-playbook upgrade 
ansible-playbook reboot 
```