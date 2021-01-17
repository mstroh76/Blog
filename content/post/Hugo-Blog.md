+++
author = "Martin Strohmayer"
title = "Hugo Blog"
date = "2021-01-04"
description = "Wie man einen Hugo Blog einrichtet..."
featured = true
tags = [
    "hugo",
    "blog",
]
categories = [
    "web"
]
image = "images/hugo.png"
thumbnail = "images/hugo.png"
+++

Hugo ist perfekt für einen statischen Blog wie diesen. So richtet amn ihn ein...
<!--more-->

## Beschreibung

[Hugo](https://de.wikipedia.org/wiki/Hugo_(Software)) ist eine freie Software (Apache-Lizenz, Version 2), die  es ermöglicht statische Webseiten zu erzeugen. Dementsprechend benötigt man für Web-Seiten die so generiert wurden, keine serverseitigen Scripts und keine Datenbank. Ein simpler kleiner Web-Space genügt für das Hosting. Die statischen Seiten können ressourcenschonenden und schnell an den Browser übermittelt werden.  
Zudem kann Hugo sehr einfach installiert und verwendet werden. Der Inhalt wir als Markdown-Datei bereitgestellt. [Markdown](https://de.wikipedia.org/wiki/Markdown) ist eine Auszeichnungssprache die viel einfacher als HTML-Dateien mit ihren Tags ist. So können selbst Anfänger ausgefeilte optisch ansprechende Webseiten erzeugen ohne Wissen über CSS, JavaScript und HTML5.
Auch für einen Blog eigent sich das System hervorragend. Dies soll hier exemplarisch beschrieben weden. 

## Installation

Zu beginn muss man sich für einen der vielen freien Themes entscheiden, und damit den Stil seiner neuen Web-Seite bzw. Blog festlegen.
Dazu sucht man am besten auf der Seite [themes.gohugo.io](https://themes.gohugo.io). 
Ich habe mich bei meinem persönlichen Blog für [Clarity](https://themes.gohugo.io/hugo-clarity/) entschieden. Wichtig ist hier zu prüfen ob man eine ausreichend aktuelle Hugo version vesitzt bzw. welche Hugo Version vom Theme vorausgesetzt wird.  


```
sudo apt-get install hugo 
hugo version
```

> Hugo Static Site Generator v0.68.3/extended linux/amd64 BuildDate: 2020-03-25T06:15:45Z

```
cd ~
hugo new site Blog
cd Blog/themes
git clone https://github.com/chipzoller/hugo-clarity themes/hugo-clarity
```


## Anpassungen/Konfiguration

Nun kann man die Beispeilkonfiguration übernehmen und entsprechend seinen Anforderungen anpassen.


```
cp -a hugo-clarity/exampleSite/* ..
cd ..
```

Mit seinem Lieblingseditor ``vim config.toml`` kann man die einsprechenden Zeilen und Einstellungen ändern.

```
baseurl: http://strohmayers.com
title: Blog
author = "Martin Stromayer"
```
 
## Inhalt erzeugen


```
mv content/post/placeholder-text.md content/post/Hugo.md
code content/post/Hugo.md
```

Nun kann man mit seinem Lieblingseditor oder z. B. auch Visual Studio Code die erste Seite schreiben. Visual Studio Code empfehle ich da es eine einfache Rechtschreibkorrektur und eine Markdown Vorschau hat.  

Zuerst definiert man die Headerdaten wie Autor, Datum und Beschreibung. Danach kann man Kategorien und Tags definieren. Diese Headerdaten werden in drei Plus Zeichen eingeschlossen. 
Dann kommt ein kleiner Vorschautext der mit ``<!--more-->`` abeschlossen wird. Danach beginnt er eigentliche Text in Markdown. 

## Web-Seite erzeugen

Nun kann man die Seite zuerst mal in einem lokalen Web-Server starten und mit einem Browser öffnen (in dem Fall verwende ich Chromium).

```
chromium-browser "http://127.0.0.1:1313/" &
hugo -D server
```