+++
author = "Martin Strohmayer"
title = "H264 CPU Hardware Encoding mit Linux"
date = "2023-10-02"
description = ""
featured = true
tags = [
    "Raspberry Pi", 
    "Video"
]
categories = [
    "Raspberry Pi"
]
image = "images/CPUvid.jpg"
thumbnail = "images/CPUvid.jpg"
+++

Inzwischen unterstützen verschiedene CPUs die Kodierung von Videos mittels Hardware Encoder. Wie geht das unter Linux und wie schnell läuft es eigentlich?
<!--more-->



Inzwischen unterschützen verschiedene CPUs die Kodierung von Videos mittels Hardware Endoder. Ich interessiere mich vor allem für den H264 Encoder um Videos bis Full-HD von MPEG nach H264 zu konvertieren. Welche Geschwindikeit ist von CPU vs CPU Encoder (HW) zu erwarten.


# Intel CPU

Intel Prozessoren unterstützen schon sehr lange die Hardware-Encoderung von Videos. Der Name dieser Erweiterung ist Intel "Quick Sync Video" und ist Teil der Intel GPU im Prozessor.
F Prozessoren bzw. Prozessoren ohne Grafikeinheit haben diese Erweiterung nicht!  
Als Konvertierungsprogramm kommt unter Linux ffmpeg zum Einsatz. Auf der Projektseite gibt es eine umfangreiche Tabelle mit unterstützten Features aller Intel Prozessorgenerationen.

https://trac.ffmpeg.org/wiki/Hardware/QuickSync


Der H264 Encoder wird bereits ab Sandy Bridge Prozessoren (also 2.Generation) unterstützt. 
Ich habe einen Laptop mit einem U4600 Haswell Prozessor (4. Generation). Dann hab ich noch einen ThinClient mit einem N3700 Braswell Prozessor. Beide sollten die Aufgabe beweltigen können.
Der U4600 Prozessor läuft mit Linux Mint 20.3 Una. Der N3700 Prozessor mit Debian 12 Bookworm.


## Installation

Zusätzlich zum ffmpeg Programm wird noch der VAAPI Treiber benötigt.  

``` 
sudo apt install vainfo ffmpeg va-driver-all
```

Nach der Installation kann man sich die VAAPI Iformationen mit vainfo ansehen.

```
vainfo
```

U4600(Haswell/Gen7.5) - Linux Mint 20.3 Una:

```
libva info: VA-API version 1.7.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_7
libva error: /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so init failed
libva info: va_openDriver() returns 1
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_6
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.7 (libva 2.6.0)
vainfo: Driver version: Intel i965 driver for Intel(R) Haswell Mobile - 2.4.0
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Simple            :	VAEntrypointEncSlice
      VAProfileMPEG2Main              :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline:	VAEntrypointVLD
      VAProfileH264ConstrainedBaseline:	VAEntrypointEncSlice
      VAProfileH264Main               :	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointEncSlice
      VAProfileH264High               :	VAEntrypointVLD
      VAProfileH264High               :	VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      :	VAEntrypointVLD
      VAProfileH264MultiviewHigh      :	VAEntrypointEncSlice
      VAProfileH264StereoHigh         :	VAEntrypointVLD
      VAProfileH264StereoHigh         :	VAEntrypointEncSlice
      VAProfileVC1Simple              :	VAEntrypointVLD
      VAProfileVC1Main                :	VAEntrypointVLD
      VAProfileVC1Advanced            :	VAEntrypointVLD
      VAProfileNone                   :	VAEntrypointVideoProc
      VAProfileJPEGBaseline           :	VAEntrypointVLD
```


N3700(Braswell/Gen8) - Debian 12 Bookworm:

```
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_17
libva error: /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so init failed
libva info: va_openDriver() returns 1
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_8
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.17 (libva 2.12.0)
vainfo: Driver version: Intel i965 driver for Intel(R) CherryView - 2.4.1
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Simple            :	VAEntrypointEncSlice
      VAProfileMPEG2Main              :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline:	VAEntrypointVLD
      VAProfileH264ConstrainedBaseline:	VAEntrypointEncSlice
      VAProfileH264Main               :	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointEncSlice
      VAProfileH264High               :	VAEntrypointVLD
      VAProfileH264High               :	VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      :	VAEntrypointVLD
      VAProfileH264StereoHigh         :	VAEntrypointVLD
      VAProfileVC1Simple              :	VAEntrypointVLD
      VAProfileVC1Main                :	VAEntrypointVLD
      VAProfileVC1Advanced            :	VAEntrypointVLD
      VAProfileJPEGBaseline           :	VAEntrypointVLD
      VAProfileJPEGBaseline           :	VAEntrypointEncPicture
      VAProfileVP8Version0_3          :	VAEntrypointVLD
      VAProfileHEVCMain               :	VAEntrypointVLD
```

Leider meldet der N3700 eine Fehler beim Aufruf mit ffmpeg. 

```
 ffmpeg: i965_encoder.c:1692: intel_enc_hw_context_init: Assertion `encoder_context->mfc_context' failed.
```

Dann habe ich die non-free Pakete in /etc/apt/sources.list hinzugefügt.
Nach einem "apt update", kann man die VAAPI non-free Treiber hinzugfügen.

```
sudo apt update
sudo apt install intel-media-va-driver-non-free i965-va-driver-shaders
```

```
vainfo
```

```
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_17
libva error: /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so init failed
libva info: va_openDriver() returns 1
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_8
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.17 (libva 2.12.0)
vainfo: Driver version: Intel i965 driver for Intel(R) CherryView - 2.4.1
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Simple            :	VAEntrypointEncSlice
      VAProfileMPEG2Main              :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline:	VAEntrypointVLD
      VAProfileH264ConstrainedBaseline:	VAEntrypointEncSlice
      VAProfileH264Main               :	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointEncSlice
      VAProfileH264High               :	VAEntrypointVLD
      VAProfileH264High               :	VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      :	VAEntrypointVLD
      VAProfilSH264MultiviewHigh      :	VAEntrypointEncSlice
      VAProfileH264StereoHigh         :	VAEntrypointVLD
      VAProfileH264StereoHigh         :	VAEntrypointEncSlice
      VAProfileVC1Simple              :	VAEntrypointVLD
      VAProfileVC1Main                :	VAEntrypointVLD
      VAProfileVC1Advanced            :	VAEntrypointVLD
      VAProfileNone                   :	VAEntrypointVideoProc
      VAProfileJPEGBaseline           :	VAEntrypointVLD
      VAProfileJPEGBaseline           :	VAEntrypointEncPicture
      VAProfileVP8Version0_3          :	VAEntrypointVLD
      VAProfileVP8Version0_3          :	VAEntrypointEncSlice
      VAProfileHEVCMain               :	VAEntrypointVLD
```

Danch sind die Einträge "VAProfileH264MultiviewHigh: VAEntrypointEncSlice" und "VAProfileH264StereoHigh : VAEntrypointEncSlice" hinzugekommen.


## Konvertierung

Beim Aufruf der Konvertierung  mit ffmpeg, muss nun der richtige Hardwareencoder angegeben werden. Das erfolgt mit dem Parameter -c:v gefolgt von Namen des Encoders.
Bei Intel ist es **h264_vaapi**. Beim Rapsberry Pi 4 ist es **h264_v4l2m2m**.  
Bei Intel muss man auch den Pfad zum VAAPI-Device angeben (/dev/dri/renderD128).
Der Input Video im Beispiel ist eine MPEG2-AVI Datei aus einer Fernsehaufzeichnung, sie muss ebenfalls angegeben werden. Das Ziel ist eine mp4-Datei.  
Wenn keine Qualitätseintstellungen angegeben werden, wird die Defaulteinstellung genommen. Je nach Encoder sollte man aber eine Datenrate oder Qualität dafür definieren.


Intel (Default Q20):  

```
time ffmpeg -hwaccel vaapi -hwaccel_output_format vaapi -hwaccel_device /dev/dri/renderD128 -i input.avi -c:v h264_vaapi output.mp4
time ffmpeg -hwaccel vaapi -hwaccel_output_format vaapi -vaapi_device /dev/dri/renderD128 -i input.avi -codec:v h264_vaapi output.mp4
```

Raspberry Pi (Average bitrate):  
```
time sudo ffmpeg -i input.avi -c:v h264_v4l2m2m -b:v 2048k output.mp4
```


## Ergebnis


**Prozessor**  |  **Speed** | **Dauer** | **Dateigröße (kiB)** | **Datenrate/Qualität**
-------------- | ---------- | --------- | -------------------- | ------------
Intel U4600    | x2,4       |           |                      | 
Intel U4600 HW | x15        |   3m7s    |  760.882 kiB         | Default: Q20  
Intel N3700    | x1,3       |           |                      | 
Intel N3700 HW | x2,6       |           |                      | 
Pi 4           | x1,2       |           |                      | 
Pi 4 HW        | x7,3       |   6m25s   | 104.191 kiB          |
Pi 4 HW        | x7,3       |   6m32s   | 639.202 kiB          | -b:v 2048k


Der Geschwindigkeits zwischen  Software und Hardwareencoder ist enorm. Bei Intel von 2,4-fache auf **15-fache**. Damit wird das 45 Minuten Video in nur 3 Minuten konvertiert.
Am langsamen N3700 Prozessor ist die Leistung des Hardwareencoders sehr gering und es wird nur eine Verdopplung geschafft. Besser als nichts, aber im Vergleich zu richtigen Mobil Prozessor viel zu langsam.  
Überraschend schnell ist auch der Raspberry Pi 4 der in weniger als 7 Minuten das Video konvertiert. Er ist zwar langsamer als Intel U4600 aber weit schneller als der vergleichbare Intel N3700 Prozessor.  

Nach dem CPU-Vergleich habe ich mir aber auch eine nVidia GPU unter Windows mit der Software Handbrake angesehen. Die konnte das Video noch schneller konvertieren als der Intel Prozessor!
Wobei das Programm die Hardwarekodierung mit meiner Desktop Skylake Intel CPU nicht beherrschte.
